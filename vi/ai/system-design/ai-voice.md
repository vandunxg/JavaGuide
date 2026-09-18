---
title: "Giải thích chi tiết công nghệ giọng nói AI: từ ASR, TTS đến triển khai Agent giọng nói realtime"
description: "Trình bày chuỗi xử lý của hệ thống giọng nói AI, bao gồm thu âm, VAD, ASR, LLM, TTS, phát streaming, xử lý ngắt lời, tối ưu latency cũng như lựa chọn API cloud, model local và phương án hybrid edge-cloud."
category: Phát triển ứng dụng AI
head:
  - - meta
    - name: keywords
      content: AI voice,ASR,TTS,VAD,realtime voice Agent,Speech to Speech,speech recognition,speech synthesis,hybrid edge-cloud,Realtime API
---

<!-- @include: @article-header.snippet.md -->

Xin chào mọi người, tôi là G.

Khi lần đầu xây dựng ứng dụng giọng nói AI, nhiều developer thường hình dung chuỗi xử lý như sau: người dùng nói, chuyển thành text, đưa cho model lớn, rồi phát câu trả lời ra.

Nghe như chỉ có ba lần gọi: **ASR -> LLM -> TTS**.

Khi đưa vào production, vấn đề xuất hiện ngay: người dùng chưa nói xong nhưng hệ thống đã nhận định sai là kết thúc; người dùng muốn ngắt lời nhưng AI vẫn tự đọc tiếp; trong phòng họp có tiếng điều hòa và tiếng bàn phím, ASR bắt đầu chuyển thành text lung tung; mạng chỉ cần chập chờn một chút là audio chiều xuống bị giật thành từng đoạn; câu trả lời dạng text trông không có vấn đề, nhưng tương tác bằng giọng nói lại giống tổng đài viên phản hồi chậm nửa nhịp.

Text Agent kết nối với microphone và loa chỉ tạo ra một Demo biết nói; hệ thống production còn phải xử lý audio realtime, model giọng nói, trạng thái hội thoại và phối hợp edge-cloud.

Phần dưới trước tiên giải thích ASR, TTS và VAD lần lượt phụ trách gì, sau đó kết hợp dự án interview-guide để thảo luận về thu âm, truyền streaming, hàng đợi phát và state machine. Cuối cùng sẽ xem API cloud, model local và phương án hybrid edge-cloud lần lượt phù hợp với trường hợp sử dụng nào.

## Giải thích thuật ngữ

Để tránh gây khó hiểu khi đọc, các thuật ngữ cốt lõi trong bài được giải thích như sau:

- **Edge** = client (browser/App), chỉ code frontend trên thiết bị của người dùng
- **Barge-in** = ngắt lời/chèn lời ngắt quãng, tức người dùng chủ động ngắt AI đang nói trong lúc model lớn phản hồi
- **Kết quả incremental** = output streaming = partial results, chỉ các kết quả trung gian mà ASR trả về realtime
- **Phương án cascade** = kiến trúc nối tiếp theo từng giai đoạn ASR + LLM + TTS
- **Native Realtime API** = interface giọng nói multimodal realtime, dạng thường gặp là audio vào, audio ra; đồng thời có thể output event text và event tool calling

## Hệ thống giọng nói AI thực sự giải quyết vấn đề gì?

Trước hết, hãy nói rõ chúng ta thực sự đang giải quyết vấn đề gì.

Voice Agent gần với một hệ thống cộng tác realtime hơn: khi người dùng nói, hệ thống phải đồng thời hoàn thành việc hiểu, tạo và phát. So với hội thoại bằng text, giọng nói có thêm một số chiều:

- **Tính realtime**: khi người dùng đang nói, hệ thống phải bắt đầu xử lý ngay, không thể đợi người dùng nói xong mới phản hồi.
- **Thông tin multimodal**: ngữ điệu, khoảng dừng và cảm xúc đều mất đi trong text.
- **Khả năng ngắt lời**: con người có thể nói chen vào nhau, máy cũng phải hỗ trợ.
- **Latency end-to-end**: chat text chậm 1 giây người dùng vẫn có thể chịu được, nhưng giọng nói chậm 1 giây sẽ khiến họ cảm thấy đối phương “không phản hồi”.

Các tương tác bằng giọng nói phổ biến trên thị trường có hai loại:

1. **Trợ lý giọng nói dạng command**: thường thấy trong nhà thông minh và điều khiển trên xe. Người dùng nói “bật điều hòa”, hệ thống ánh xạ giọng nói vào intent và command thiết bị đã định nghĩa trước. Các sản phẩm như Siri, Xiao Ai cũng đang tích hợp khả năng hỏi đáp mở, nên không thể đơn giản xếp vào menu cố định.
2. **Voice Agent dùng model lớn**: có thể hiểu câu hỏi mở, gọi tool và hội thoại nhiều lượt liên tục. Bạn hỏi “giúp tôi xem API lần trước bị timeout là do đâu”, nó phải hiểu intent, truy xuất context, tạo câu trả lời, đồng thời dùng giọng nói để xác nhận qua lại với bạn.

Trọng tâm engineering của hai loại sản phẩm này khác nhau rất nhiều. Phần dưới thảo luận việc triển khai Voice Agent dùng model lớn trong production.

## Nhận dạng giọng nói (ASR) chuyển âm thanh thành text như thế nào?

ASR (Automatic Speech Recognition) trông như chỉ là “audio vào, text ra”, nhưng phía sau ít nhất bao gồm ba phán đoán:

1. Đoạn audio này nói từ nào.
2. Các từ đó được tách thành từ và câu như thế nào.
3. Dấu câu, số, tiếng Anh và technical term được chuẩn hóa ra sao.

Ví dụ người dùng nói “giúp tôi tra virtual thread của Java 21”, ASR phải đồng thời nhận dạng tiếng Việt, tiếng Anh, số và technical term. Nếu nhận dạng thành “giúp tôi tra hai mươi mốt của Java virtual thread”, dù LLM phía sau mạnh đến đâu cũng phải đoán lại rất lâu.

### Ba hướng công nghệ của ASR

| Loại                          | Phương án tiêu biểu                                                                                                                                                              | Ưu điểm                                                                                                                 | Nhược điểm                                                                                                        | Trường hợp sử dụng                                                             |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| API cloud                     | OpenAI Audio Transcriptions（`gpt-4o-transcribe`、`gpt-4o-mini-transcribe`、`whisper-1`、`gpt-4o-transcribe-diarize`）、Azure Speech、Google Speech、Deepgram、Alibaba Cloud ASR | Tích hợp nhanh, hỗ trợ nhiều ngôn ngữ, chi phí vận hành thấp                                                            | Bị giới hạn bởi chi phí, network latency và compliance dữ liệu                                                    | CSKH, chuyển biên bản cuộc họp, voice assistant nhẹ                            |
| Model open source tổng quát   | Whisper、faster-whisper、Whisper.cpp、FunASR                                                                                                                                     | Có thể deploy local, khả năng kiểm soát cao, hỗ trợ private deployment; faster-whisper có thể kết hợp Silero VAD để lọc | Phải tự tối ưu engineering cho realtime; Whisper turbo không được train cho translation, hiệu quả translation kém | Chuyển biên bản private deployment, subtitle offline, mạng nội bộ doanh nghiệp |
| Model tùy chỉnh theo lĩnh vực | ASR chuyên dụng cho tài chính, y tế, xe                                                                                                                                          | Thích ứng tốt hơn với technical term và giọng vùng miền                                                                 | Chi phí chuẩn bị data và training cao                                                                             | Trường hợp vertical có tần suất cao, vocabulary nghiệp vụ đặc thù              |

**Bổ sung**:

- `gpt-4o-transcribe-diarize` của OpenAI hỗ trợ nhãn speaker, phù hợp với các trường hợp nhiều người như chuyển biên bản cuộc họp. Hiện model này chỉ dùng cho `/v1/audio/transcriptions`, không hỗ trợ Realtime API; khi audio dài hơn 30 giây, cần cấu hình `chunking_strategy`; model cũng không hỗ trợ `prompt`, `logprobs`, `timestamp_granularities[]`. Nếu không cần nhãn speaker, nên ưu tiên `gpt-4o-transcribe`, `gpt-4o-mini-transcribe` hoặc `whisper-1`.
- Whisper turbo (large-v3-turbo) là bản tối ưu inference của large-v3, tốc độ nhanh nhưng **không được train cho tác vụ translation**. Khi chạy `--task translate`, model sẽ output ngôn ngữ gốc thay vì tiếng Anh; nếu cần translation, hãy dùng medium hoặc large.
- Cần phân biệt chuyển biên bản realtime với chuyển biên bản file ghi âm. Tài liệu hiện tại của OpenAI đặt chức năng chuyển biên bản realtime latency thấp trong Realtime transcription, model là `gpt-realtime-whisper`; các tác vụ như upload file và tách speaker dùng Audio Transcriptions.

**Gợi ý lựa chọn**: nếu nhu cầu cốt lõi của bạn là “hội thoại realtime”, đừng chỉ nhìn WER offline (Word Error Rate, tỷ lệ lỗi từ). Bạn nên quan tâm hơn đến:

- **Latency đoạn đầu tiên**: thời gian từ lúc người dùng nói xong đến khi nhìn thấy từ đầu tiên
- **Độ ổn định của kết quả incremental**: có thể realtime nhìn thấy tiến độ nhận dạng hay không
- **Độ chính xác của phát hiện endpoint**: có thể xác định chính xác người dùng đã nói xong hay chưa
- **Hiệu quả trong môi trường nhiễu**: độ chính xác khi ở xa hoặc có nhiều người nói
- **Khả năng hotword**: có thể nhận dạng vocabulary riêng của nghiệp vụ hay không

### Khác biệt giữa streaming ASR và non-streaming ASR

Với hội thoại realtime yêu cầu cao về latency ký tự đầu tiên và ngắt lời tự nhiên, thường dùng streaming ASR; những trường hợp có lượt nói rõ ràng và audio ngắn cũng có thể dùng nhận dạng non-streaming. Khác biệt chính giữa hai loại:

- **Non-streaming ASR**: đợi người dùng nói xong một đoạn rồi nhận dạng cả đoạn. Latency = thời lượng nói + thời gian nhận dạng.
- **Streaming ASR**: vừa nói vừa nhận dạng, ngay khi người dùng vừa dứt lời đã có thể lấy kết quả. Latency ≈ thời gian phát hiện endpoint + thời gian nhận dạng realtime.

Dự án interview-guide dùng **qwen3-asr-flash-realtime của Alibaba Cloud DashScope**. Cách tích hợp này liên tục append audio qua WebSocket, VAD phía server phụ trách xác định thời điểm submit một lượt nhận dạng:

```java
// QwenAsrService.java
OmniRealtimeConfig config = OmniRealtimeConfig.builder()
    .modalities(Collections.singletonList(OmniRealtimeModality.TEXT))
    .enableTurnDetection(true)  // Bật VAD phía server
    .turnDetectionType("server_vad")
    .turnDetectionSilenceDurationMs(400)  // Im lặng 400 ms thì xác định người dùng đã nói xong
    .transcriptionConfig(transcriptionParam)
    .build();
```

Ưu điểm của VAD phía server là client không phải tự triển khai đầy đủ logic phát hiện hoạt động giọng nói; cái giá phải trả cũng được ghi trực tiếp trong parameter: `turnDetectionSilenceDurationMs(400)` nghĩa là chỉ coi một câu đã kết thúc sau khi im lặng liên tục 400 ms. Khoảng giá trị trong tài liệu DashScope là 200-6000 ms; giá trị càng thấp thì response càng nhanh nhưng càng dễ cắt mất khoảng dừng tự nhiên; giá trị càng cao thì việc ngắt câu càng thận trọng và latency cũng tăng. VAD phía edge và VAD phía server có thể kết hợp, nhưng không phải kiến trúc cố định: khi cần ngắt lời latency thấp, có thể để edge báo trước `speech_start`, rồi server kết hợp event ASR và VAD để xác nhận kết thúc lượt; khi client nhẹ hoặc môi trường mạng ổn định, cũng có thể chỉ dùng phát hiện endpoint phía server.

## Tổng hợp giọng nói (TTS) chuyển text thành âm thanh như thế nào?

TTS (Text To Speech) phụ trách tổng hợp audio từ câu trả lời của model. Nó trông như layer output, nhưng thực tế ảnh hưởng rất lớn đến cảm nhận của người dùng về toàn bộ Agent.

Cùng một câu “để tôi tra giúp bạn”, sự khác biệt giữa các TTS có thể thể hiện ở:

- Phải đợi bao lâu mới có audio đầu tiên
- Âm sắc có tự nhiên không, câu dài có thở như người thật không
- Số, code và viết tắt tiếng Anh có được đọc chính xác không
- Có hỗ trợ điều khiển cảm xúc, tốc độ nói, khoảng dừng và cao độ không

### Tiến hóa công nghệ TTS

TTS truyền thống đi qua nhiều bước:

```
Chuẩn hóa text -> Phân tích text -> Acoustic model -> Vocoder -> Output waveform
```

Neural TTS, neural audio codec và generative speech model đang giảm bớt các module thủ công trong pipeline truyền thống. Phương pháp modeling của VALL-E, Fish Speech và CosyVoice không giống nhau; chất lượng âm thanh, latency, khả năng streaming và chi phí deploy cũng không thể chỉ so sánh theo một tiêu chí “end-to-end”. Với Voice Agent realtime, ngoài chất lượng âm thanh của từng câu, còn phải xem latency audio đầu tiên, có thể phát streaming hay không và xử lý state sau khi bị ngắt như thế nào.

Nếu phải đợi toàn bộ text được tạo xong mới tổng hợp, cảm nhận của người dùng sẽ rất chậm. Nếu có thể tổng hợp streaming theo câu ngắn hoặc thậm chí theo token, trải nghiệm audio đầu tiên sẽ tốt hơn nhiều.

### Hai hướng của TTS realtime

| Loại               | Phương án tiêu biểu                                                                                   | Đặc điểm                                                                                                                                              |
| ------------------ | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| TTS realtime cloud | OpenAI Speech、Alibaba Cloud qwen3-tts-flash-realtime / Qwen-TTS-Realtime、Azure TTS、ElevenLabs      | Output streaming, hỗ trợ tổng hợp realtime                                                                                                            |
| TTS local          | piper1-gpl（GPL-3.0，Piper gốc đã bị archive）、Fish Speech（Fish Audio Research License）、CosyVoice | Khả năng kiểm soát cao, phù hợp với trường hợp offline; trước khi dùng commercial cần kiểm tra từng license của code, model weights và voice resource |

interview-guide cũng dùng TTS realtime của Alibaba Cloud, tổng hợp audio qua WebSocket. Trong ví dụ Java SDK hiện tại của DashScope, tên model được khuyến nghị là `qwen3-tts-flash-realtime`, còn wrapper class trong dự án vẫn có tên `QwenTtsRealtime`:

```java
// QwenTtsService.java
QwenTtsRealtimeConfig config = QwenTtsRealtimeConfig.builder()
    .voice(voice)  // Chọn voice
    .responseFormat(QwenTtsRealtimeAudioFormat.PCM_24000HZ_MONO_16BIT)
    .mode("commit")  // Chế độ commit
    .languageType(languageType)
    .speechRate(speechRate)
    .volume(volume)
    .build();

// Gửi text, nhận audio chunk realtime
qwenTtsRealtime.appendText(text);
qwenTtsRealtime.commit();
```

Đoạn code này dùng mode `commit`; sau khi client append text, nó chủ động gọi `commit()` để kích hoạt việc tổng hợp. Tài liệu DashScope cũng cung cấp mode `server_commit`, trong đó server xác định thời điểm commit; sự đánh đổi giữa latency và tính hoàn chỉnh của câu sẽ khác nhau.

## VAD điều khiển lượt hội thoại như thế nào?

VAD (Voice Activity Detection, phát hiện hoạt động giọng nói) là component thường bị bỏ qua, nhưng ảnh hưởng cực lớn đến trải nghiệm.

VAD không nhận dạng nội dung lời nói, thường output xác suất một đoạn audio ngắn có chứa giọng nói hoặc event bắt đầu, kết thúc giọng nói. Ứng dụng dựa vào đó để xác định:

- Người dùng đã bắt đầu nói chưa?
- Người dùng đã nói xong chưa?
- Hoạt động giọng nói hiện tại đã đủ để trigger một lần ngắt lời hoặc submit chưa.

VAD thông thường không thể tự xác định âm thanh đến từ người dùng, người bên cạnh, nhạc hay audio phát lại từ loa. Echo do hệ thống phát cần được xử lý bằng AEC hoặc tín hiệu tham chiếu phát; trường hợp nhiều người còn có thể cần tách speaker, voiceprint hoặc VAD hướng đến speaker mục tiêu. Cách nói thực tế của người dùng cũng làm tăng độ khó của việc phát hiện endpoint:

- Có thể dừng giữa câu: “Vấn đề này... tôi muốn hỏi...”
- Có các phản hồi ngắn: “ừm”, “đúng”, “không phải”
- Vừa suy nghĩ vừa nói, âm lượng lúc to lúc nhỏ
- Bên cạnh có thể có người nói, loa cũng có thể đang phát giọng AI

**VAD phía edge hay VAD phía server?**

| Loại            | Phương án tiêu biểu                                                                                                  | Ưu điểm                                                        | Nhược điểm                                                                                   |
| --------------- | -------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| VAD phía edge   | WebRTC VAD、Silero VAD、@ricky0123/vad-web                                                                           | Response nhanh, không tiêu tốn resource phía server            | Cần deploy model trên client, phải tự điều chỉnh threshold và trường hợp nhiễu               |
| VAD phía server | server_vad của DashScope ASR、OpenAI Realtime turn detection、phát hiện endpoint tích hợp sẵn trong một số ASR cloud | Logic client đơn giản, tích hợp chặt hơn với service nhận dạng | Tăng tải server, có network latency, strategy ngắt câu bị ràng buộc bởi interface của vendor |

> ⚠️ **Không thể chỉ nhìn accuracy offline của VAD**: audio ngắn (<1 giây, như “ừm”, “đúng”, “không phải”), lời nói chen âm lượng nhỏ, giọng nói từ xa và echo loa đều khiến hiệu quả online của VAD khác rất nhiều so với tập thử nghiệm. README của faster-whisper cũng mô tả strategy mặc định của Silero VAD là khá thận trọng: mặc định chỉ loại bỏ đoạn im lặng dài hơn 2 giây. Trong Voice Agent, nếu coi VAD là tiêu chí duy nhất để ngắt lời, rất dễ bỏ sót các phản hồi ngắn.

Frontend của interview-guide dùng **@ricky0123/vad-web**, đây là VAD phía edge dựa trên ONNX:

```typescript
// AudioRecorder.tsx
const vadInstance = await window.vad.MicVAD.new({
  getStream: async () => stream,
  onnxWASMBasePath: "https://cdn.jsdelivr.net/npm/onnxruntime-web@1.22.0/dist/",
  baseAssetPath: "https://cdn.jsdelivr.net/npm/@ricky0123/vad-web@0.0.29/dist/",
  onSpeechStart: () => {
    onSpeechStart?.(); // Người dùng bắt đầu nói
  },
  onSpeechEnd: () => {
    onSpeechEnd?.(); // Người dùng nói xong
  },
});
```

Sau khi VAD phía edge trigger `onSpeechEnd`, không nên submit vô điều kiện. Có thể thêm một khoảng thời gian xác nhận im lặng có thể cấu hình, hoặc kết hợp với event transcription cuối cùng từ server để tránh coi việc người dùng tạm dừng giữa chừng là kết thúc. Thời gian xác nhận phải được điều chỉnh qua sample online theo ngôn ngữ, nhịp nói và mục tiêu latency; không thể coi 300-500 ms là threshold dùng chung.

Theo tôi: **đừng chỉ dùng VAD như một công tắc, nó nên output một nhóm tín hiệu điều khiển hội thoại**. Ví dụ:

- `speech_start`: người dùng bắt đầu nói
- `speech_end`: phát hiện hoạt động giọng nói kết thúc (có thể kèm confidence)
- `maybe_barge_in`: có thể người dùng đang ngắt lời
- `non_speech`: chunk hiện tại không phát hiện hoạt động giọng nói

## Một hội thoại bằng giọng nói hoàn chỉnh chạy như thế nào?

Trước hết hãy đặt cả chuỗi xử lý cạnh nhau, như vậy mới dễ hiểu latency, ngắt lời và phối hợp edge-cloud ở phần sau.

Một hội thoại Voice Agent thường đi qua các bước sau:

1. Thu audio: microphone thu audio raw
2. Preprocessing: AEC khử echo, NS khử noise, AGC điều chỉnh gain
3. Phát hiện VAD: xác định người dùng có đang nói và đã nói xong chưa
4. Upload audio: gửi audio đã xử lý lên server
5. Chuyển biên bản ASR: chuyển audio thành text (output kết quả incremental theo streaming)
6. Lắp ghép context: nối system instruction, lịch sử hội thoại và định nghĩa tool
7. Inference LLM: hiểu intent, tạo response và gọi tool khi cần
8. Tổng hợp TTS: chuyển text response thành audio (output audio chunk theo streaming)
9. Audio chiều xuống: client vừa nhận vừa phát
10. Ghi lại state: ghi lại hội thoại hiện tại, chuẩn bị context cho lượt tiếp theo

Trong chuỗi xử lý giọng nói realtime, một phần công việc có thể hoàn tất trước khi người dùng nói xong.

Hệ thống tốt sẽ cố gắng thực hiện sớm những việc có thể làm trước:

- Khi người dùng vừa bắt đầu nói, trước tiên load session state và tool definition
- Sau khi ASR xuất hiện prefix ổn định, thực hiện dự đoán intent trước
- Khi LLM output câu ngắn đầu tiên, TTS lập tức bắt đầu tổng hợp
- Khi tool calling chậm, trước tiên phát một câu chuyển tiếp tự nhiên

Cách làm rất trực tiếp: khởi động sớm các khâu có thể chạy song song, dùng output streaming để chia nhỏ thời gian chờ.

## Vì sao voice realtime khó hơn hội thoại text nhiều như vậy?

Điểm khó của hội thoại bằng giọng nói không nằm ở một model nào đó, mà ở việc toàn bộ chuỗi xử lý đều bị ràng buộc bởi realtime.

### Khó khăn một: ngân sách latency rất chặt

Chat text chậm 1 giây, người dùng thường vẫn có thể chịu được. Hội thoại bằng giọng nói chậm 1 giây sẽ khiến người dùng rõ ràng cảm thấy đối phương “không phản hồi”.

Latency của một lượt tương tác bằng giọng nói đến từ các khâu sau:

| Khâu                   | Thời gian thường gặp                                     | Hướng tối ưu                                          |
| ---------------------- | -------------------------------------------------------- | ----------------------------------------------------- |
| Thu và encode          | Kích thước audio frame, buffer browser                   | Thu frame nhỏ, giảm buffer không cần thiết            |
| Phát hiện endpoint VAD | Đợi xác nhận im lặng người dùng đã nói xong              | Threshold im lặng động, submit nhanh câu ngắn         |
| ASR                    | Upload audio, decode, ổn định transcription incremental  | Streaming ASR, hotword, preprocessing phía edge       |
| LLM                    | Latency token đầu tiên, tool calling, context quá dài    | Cache Prompt, response ngắn, tool bất đồng bộ         |
| TTS                    | Tổng hợp audio đầu tiên, tách câu dài, inference vocoder | Tổng hợp streaming theo câu, warm-up voice            |
| Phát                   | Network jitter, decode, buffer player                    | Vừa nhận vừa phát, kiểm soát playback queue và buffer |

Nếu mỗi đoạn tăng thêm 200 ms, cả lượt hội thoại sẽ lập tức trở thành “chậm nửa nhịp”.

Vì vậy, tối ưu voice realtime phải theo dõi latency P95/P99 end-to-end, thay vì chỉ đưa một component nào đó lên giới hạn lý thuyết. Người dùng cảm nhận toàn bộ chuỗi xử lý, không phải benchmark của một model.

### Khó khăn hai: xử lý ngắt lời không phải nút pause

Voice Agent bắt buộc phải hỗ trợ **Barge-in (ngắt lời)**.

Khi người dùng nói “đợi một chút, không phải ý này”, hệ thống cần đồng thời làm nhiều việc:

1. Nhận dạng đây là người dùng đang nói, không phải background noise hoặc echo từ loa
2. Lập tức dừng playback queue local, không thể tiếp tục phát hết response cũ
3. Cancel stream LLM và TTS vẫn đang được generate ở server
4. Ghi nội dung đã phát, chưa phát và bị ngắt vào conversation state
5. Dùng audio mới của người dùng để bắt đầu lượt hiểu tiếp theo

Nhiều hệ thống ngắt lời thất bại không nhất thiết vì VAD không chính xác; vấn đề thường gặp hơn là state machine chưa mô tả rõ semantics của cancel. Ví dụ player đã dừng nhưng TTS phía server vẫn đang stream; LLM đã dừng nhưng trong history đã ghi response chưa phát là “đã nói”.

Cách làm của interview-guide là:

```typescript
// VoiceInterviewPage.tsx
const handleAudioData = (audioData: string) => {
  // Khi AI đang phát thì dừng gửi audio, tránh nhận dạng giọng của chính mình
  if (isAiSpeakingRef.current) {
    return;
  }
  if (wsRef.current && wsRef.current.isConnected()) {
    wsRef.current.sendAudio(audioData);
  }
};
```

Frontend dùng `isAiSpeakingRef` để đánh dấu AI có đang nói hay không, khi đang nói thì dừng gửi audio. Backend nhận message `control` để cancel việc generate.

### Khó khăn ba: môi trường nhiễu phức tạp hơn môi trường test rất nhiều

Voice Demo thường chạy trong văn phòng yên tĩnh, còn production có thể là:

- Trong xe, nhà máy, trung tâm thương mại, ga tàu điện
- Microphone ở xa, người dùng cách thiết bị hai ba mét
- Nhiều người nói đồng thời
- Người dùng bật loa ngoài, giọng AI lại bị microphone thu vào

Điều này ảnh hưởng đến toàn bộ chuỗi xử lý:

- VAD coi noise là giọng người, gây trigger nhầm
- ASR chuyển giọng người trong background thành text, làm nhiễu intent của người dùng
- Audio TTS phát ra bị microphone thu lại, gây self-interruption

Frontend interview-guide bật 3 option preprocessing audio phổ biến qua `getUserMedia`:

```typescript
const stream = await navigator.mediaDevices.getUserMedia({
  audio: {
    echoCancellation: true, // AEC: khử echo từ loa
    noiseSuppression: true, // NS: giảm background noise
    autoGainControl: true, // AGC: tự động điều chỉnh gain, giúp âm lượng ổn định hơn
    sampleRate: 16000,
  },
});
```

Ba parameter này có thể giải quyết một phần vấn đề, nhưng không thể kỳ vọng chúng bao phủ mọi trường hợp. Browser chỉ tiếp nhận constraint và cố gắng đáp ứng, hiệu quả cụ thể phụ thuộc vào browser, thiết bị, microphone và môi trường phát; AEC có hiệu quả hạn chế trong trường hợp echo mạnh, NS cũng có thể làm mất một phần giọng người dùng. Nếu muốn xây dựng phương án hardware hoặc App, preprocessing audio phía edge sẽ trở thành một khoản đầu tư engineering rất thực tế.

### Khó khăn bốn: context không chỉ là lịch sử text

Context của Text Agent chủ yếu là message history. Context của Voice Agent nhiều hơn:

- Người dùng hiện có đang nói hay không
- Response trước đã phát đến đâu
- Người dùng đang hỏi bình thường hay đang ngắt lời
- Text incremental của ASR đã ổn định hay chưa
- Ngữ điệu của người dùng là nghi vấn, phủ định, do dự hay thiếu kiên nhẫn
- Hiện có tool calling nào đang thực thi hay không

Nếu chỉ đưa text ASR cuối cùng cho LLM, rất nhiều thông tin sẽ bị mất.

Ví dụ người dùng nói “không phải... tôi muốn nói đơn hàng tháng trước”, text thể hiện được việc sửa lại, nhưng không thể hiện họ đang ngắt AI; nếu hệ thống không biết response trước đã phát đến đâu thì rất khó biết người dùng đang phủ định câu nào.

interview-guide phân biệt các state khác nhau bằng WebSocket message type:

```typescript
// voiceInterview.ts
export interface WebSocketSubtitleMessage {
  type: "subtitle";
  text: string;
  isFinal: boolean; // true chỉ có nghĩa đoạn ASR transcription này đã kết thúc hoặc ổn định
}

export interface WebSocketAudioResponseMessage {
  type: "audio";
  data: string; // Audio Base64
  text: string; // Text tương ứng
}

export interface WebSocketControlMessage {
  type: "control";
  action: string; // 'submit' | 'cancel' | 'pause'
  data?: Record<string, unknown>;
}
```

`isFinal` là state của event ASR, không có nghĩa người dùng đã click hoặc nói “submit”. Nếu sản phẩm dùng submit thủ công, cần dùng event độc lập `control: submit` để xác nhận; với mode tự động theo lượt, việc có submit hay không phải do endpoint detection, event cuối cùng của ASR và business state cùng quyết định, không thể dùng một boolean để biểu đạt hai tầng ngữ nghĩa.

### Khó khăn năm: echo gây ngắt lời nhầm

Âm thanh AI phát ra bị microphone thu vào, VAD hoặc ASR có thể nhận định sai là người dùng đang nói, khiến AI tự ngắt chính mình.

Cách làm hiện tại của interview-guide là:

```typescript
if (isAiSpeakingRef.current) {
  return; // Khi AI nói thì dừng gửi audio
}
```

Phương án “âm thầm loại bỏ” này có thể tránh self-interruption, nhưng cũng chặn lời người dùng nói chen trong lúc AI đang nói.

Phương án tinh vi hơn thường làm như sau:

- Khi AI nói, tiếp tục nhận audio nhưng không gửi tới ASR
- Chạy VAD phía edge trên audio đã qua xử lý AEC thay vì audio microphone raw
- Kết hợp echo reference, năng lượng của các frame liên tiếp, confidence của VAD và state của playback queue để phán đoán có thực sự là người dùng đang nói chen hay không

### Khó khăn sáu: năng lực phía edge quyết định giới hạn trải nghiệm

Nhiều team đưa toàn bộ capability lên cloud, kết quả là trải nghiệm sụp đổ rất nhanh trong môi trường mạng yếu.

Edge ít nhất nên đảm nhận các trách nhiệm sau:

- Thu microphone và preprocessing audio
- VAD hoặc phát hiện ngắt lời nhẹ
- Buffer playback và cancel playback
- Thông báo và reconnect khi mạng ngắt

Model cloud quyết định giới hạn trên, còn engineering phía edge quyết định giới hạn dưới. Điều này đặc biệt đúng trong hệ thống giọng nói.

## Xem interview-guide để hiểu Voice Agent bản cơ bản được triển khai thế nào

Sau đây lấy dự án interview-guide làm ví dụ để xem một Voice Agent phỏng vấn bản cơ bản chạy như thế nào.

### Kiến trúc tổng thể

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (React)                     │
├─────────────────────────────────────────────────────────────┤
│  AudioRecorder        WebSocket         VoiceInterviewPage   │
│  - getUserMedia       - sendAudio       - Quản lý state      │
│  - AudioWorklet       - sendControl     - Submit thủ công    │
│  - Phát hiện VAD      - Message control - Phát theo chunk   │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                     Backend (Spring Boot)                    │
├─────────────────────────────────────────────────────────────┤
│  VoiceInterviewWebSocketHandler                             │
│  - Quản lý session (tạo, pause, resume, kết thúc)            │
│  - Đồng bộ state ASR ready / reconnect                       │
│  - Route audio tới ASR, trigger LLM sau submit thủ công      │
│  - Output stream theo câu của LLM, TTS vừa tổng hợp vừa push  │
├─────────────────────────────────────────────────────────────┤
│  QwenAsrService          DashscopeLlmService      QwenTtsService │
│  - qwen3-asr-flash-      - qwen-max / qwen-plus   - qwen-tts-  │
│    realtime              - Hỗ trợ tool calling      realtime     │
└─────────────────────────────────────────────────────────────┘
```

### Frontend: thu audio và VAD

Core của frontend là component `AudioRecorder`. Nó thực hiện những việc sau:

**Bước một, lấy quyền microphone và cấu hình parameter audio:**

```typescript
const stream = await navigator.mediaDevices.getUserMedia({
  audio: {
    echoCancellation: true,
    noiseSuppression: true,
    autoGainControl: true,
    sampleRate: 16000, // ASR cần 16 kHz
  },
});
```

**Bước hai, khởi tạo VAD phía edge:**

```typescript
const vadInstance = await window.vad.MicVAD.new({
  getStream: async () => stream,
  onSpeechStart: () => {
    onSpeechStart?.(); // Trigger callback
  },
  onSpeechEnd: () => {
    onSpeechEnd?.();
  },
});
await vadInstance.start();
```

**Bước ba, dùng AudioWorklet để thu audio theo chunk:**

Chỉ `onSpeechEnd` của VAD cho biết hoạt động giọng nói có thể đã kết thúc, audio vẫn phải được chia chunk và gửi tới server. Cách triển khai của interview-guide là:

```typescript
await audioContext.audioWorklet.addModule("/audio-worklet/pcm-processor.js");

const workletNode = new AudioWorkletNode(audioContext, "pcm-processor");
workletNode.port.onmessage = (event) => {
  if (!recordingActiveRef.current) {
    return;
  }
  const base64 = arrayBufferToBase64(event.data as ArrayBuffer);
  onAudioData(base64); // Int16 PCM 200 ms, gửi tới ASR backend
};

source.connect(workletNode);
workletNode.connect(gainNode);
gainNode.connect(audioContext.destination);
```

`pcm-processor.js` chạy trên audio rendering thread, phụ trách resample audio Float32 đầu vào của browser thành 16 kHz, Int16 PCM và gửi lại main thread qua `postMessage`, mỗi chunk 200 ms. So với `ScriptProcessorNode` đã deprecated, `AudioWorkletNode` không dồn việc xử lý audio lên main thread của UI, nên latency và rủi ro giật lag thấp hơn.

Ở đây có một lựa chọn thiết kế: **tại sao không đợi VAD trigger `onSpeechEnd` rồi mới gửi audio?**

Vì VAD có latency; nếu đợi nó xác nhận người dùng đã nói xong rồi mới bắt đầu gửi audio, sẽ phải đợi thêm một khoảng thời gian xác nhận im lặng. Cách hợp lý hơn là liên tục gửi audio theo chunk; VAD trigger `onSpeechEnd` chỉ thông báo cho backend rằng “đoạn này có thể đã kết thúc, có thể chuẩn bị submit cho LLM”.

Tuy nhiên, phỏng vấn bằng giọng nói của interview-guide không dùng “phát hiện im lặng thì tự động submit”. Cách làm của nó là **ASR liên tục transcription, người dùng click submit thủ công**. Điều này tránh việc hệ thống trả lời trước khi ứng viên tạm dừng giữa chừng, đồng thời giải quyết trải nghiệm “câu nói phía sau ghi đè câu trả lời phía trước”: frontend chỉ dùng kết quả ASR làm bản nháp câu trả lời, còn việc bước vào lượt phỏng vấn tiếp theo do message control `submit` quyết định.

### Frontend: phát audio

interview-guide dùng hai mode phát audio:

**Mode một: HTMLAudioElement (trường hợp đơn giản):**

```typescript
// VoiceInterviewPage.tsx
const onAudioResponse = (audioData: string, text: string) => {
  if (audioData && audioData.length > 0) {
    setAiAudio(audioData); // Đặt src, trigger autoplay
    setAiText(text);
    setAiSpeaking(true);

    // Đặt watchdog timeout để tránh playback audio bị treo bất thường
    const durationMs = estimateWavDurationMs(audioData);
    audioPlaybackWatchdogRef.current = setTimeout(
      finishAiPlayback,
      Math.min(Math.max(durationMs + 1500, 4000), 60_000),
    );
  }
};
```

**Mode hai: Phát theo chunk bằng AudioContext (kiểm soát chi tiết hơn):**

```typescript
// Xử lý chunk
const handleAudioChunk = (
  base64Wav: string,
  _index: number,
  isLast: boolean,
) => {
  // 1. Decode WAV
  const binaryStr = atob(base64Wav);
  const bytes = new Uint8Array(binaryStr.length);
  for (let i = 0; i < binaryStr.length; i++) {
    bytes[i] = binaryStr.charCodeAt(i);
  }

  // Xử lý header WAV 44 byte và PCM 24 kHz, mono, 16-bit theo convention của dự án.
  // Nếu server có thể trả về format WAV khác, trước tiên phải parse header WAV, không được hard-code.
  const pcmOffset = 44;
  const pcmData = new Int16Array(
    bytes.buffer,
    pcmOffset,
    (bytes.length - pcmOffset) / 2,
  );
  const float32 = new Float32Array(pcmData.length);
  for (let i = 0; i < pcmData.length; i++) {
    float32[i] = pcmData[i] / 32768;
  }

  const audioBuffer = ctx.createBuffer(1, float32.length, 24_000);
  audioBuffer.copyToChannel(float32, 0);

  // 2. Đưa vào playback queue
  chunkQueueRef.current.push(audioBuffer);
  if (!isChunkPlayingRef.current) {
    playNextChunk();
  }

  // 3. Sau chunk cuối hoặc audio_complete từ server, đợi queue phát xong
  if (isLast) {
    scheduleChunkDrainCompletion();
  }
};

// Phát chunk tiếp theo
const playNextChunk = () => {
  if (chunkQueueRef.current.length === 0) {
    isChunkPlayingRef.current = false;
    return;
  }
  const buffer = chunkQueueRef.current.shift()!;
  const source = ctx.createBufferSource();
  source.buffer = buffer;
  source.connect(ctx.destination);
  source.onended = () => playNextChunk();
  source.start(0);
};
```

Ưu điểm của phát theo chunk là có thể bắt đầu phát nhanh hơn, không cần đợi load toàn bộ audio file. Cái giá phải trả cũng rất rõ ràng: phải tự quản lý queue, thứ tự, cancel và semantics của “chunk cuối”.

Trong implementation mới, sau khi gửi xong toàn bộ TTS chunk, server còn push thêm một message control `audio_complete`. Nhờ vậy frontend không còn phụ thuộc vào việc một audio chunk nào đó nhất định phải có `isLast=true`; ngay cả khi một câu TTS tổng hợp thất bại, vẫn có thể kết thúc chính xác state “interviewer đang nói” sau khi phát xong các chunk đã thành công.

> ⚠️ **Lưu ý**: browser yêu cầu AudioContext phải được tạo hoặc resume sau khi người dùng tương tác (autoplay policy). Nếu tạo AudioContext khi page load, phần lớn browser sẽ đặt nó ở state `suspended`. Nên gọi `audioContext.resume()` khi người dùng click button “bắt đầu phỏng vấn” để đảm bảo playback hoạt động bình thường.

### Backend: quản lý WebSocket session

Backend quản lý lifecycle của session thông qua `VoiceInterviewWebSocketHandler`:

```java
// VoiceInterviewWebSocketHandler.java
public class VoiceInterviewWebSocketHandler {
    // Session state: idle -> listening -> thinking -> speaking -> completed
    // Hỗ trợ: pause (tạm dừng), resume (tiếp tục), end (kết thúc)

    // Nhận audio từ client
    public void handleAudioMessage(String sessionId, String audioBase64) {
        asrService.sendAudio(sessionId, decodeBase64(audioBase64));
    }

    // Nhận message control từ client
    public void handleControlMessage(String sessionId, String action, Map data) {
        switch (action) {
            case "submit" -> llmService.triggerResponse(sessionId, data);
            case "cancel" -> cancelCurrentGeneration(sessionId);
            case "pause" -> pauseSession(sessionId);
        }
    }
}
```

State machine của session trong interview-guide:

| State       | Ý nghĩa                                               | Có thể chuyển đến |
| ----------- | ----------------------------------------------------- | ----------------- |
| IN_PROGRESS | Phỏng vấn đang diễn ra                                | PAUSED, COMPLETED |
| PAUSED      | Tạm dừng (người dùng rời page hoặc chủ động tạm dừng) | IN_PROGRESS       |
| COMPLETED   | Phỏng vấn kết thúc                                    | -                 |

Cơ chế pause/resume rất hữu ích. Ví dụ người dùng nghe điện thoại hoặc chuyển tab, có thể pause phỏng vấn rồi tiếp tục liền mạch khi quay lại.

### Backend: service ASR

Service ASR phía backend đóng gói interface của Alibaba Cloud DashScope:

```java
// QwenAsrService.java
public void startTranscription(
    String sessionId,
    Consumer<String> onFinal,
    Consumer<String> onPartial,
    Runnable onReady,
    Consumer<Throwable> onError
) {
    // 1. Tạo session và thiết lập kết nối WebSocket
    OmniRealtimeConversation conversation = new OmniRealtimeConversation(param, callback);
    conversation.connect();

    // 2. Cấu hình: bật VAD phía server, 400 ms im lặng thì xác định kết thúc
    OmniRealtimeConfig config = OmniRealtimeConfig.builder()
        .enableTurnDetection(true)
        .turnDetectionSilenceDurationMs(400)
        .build();

    // 3. Gửi cấu hình session. Ở đây không được lập tức đổi state local thành ready
    conversation.updateSession(config);
}

// 4. Chỉ mở upload audio sau khi callback nhận event session.updated từ server
public void onSessionUpdated(String sessionId) {
    AsrSession asrSession = sessions.get(sessionId);
    asrSession.markReady();
    asrSession.getOnReady().run(); // Thông báo asr_ready cho frontend
}

public void sendAudio(String sessionId, byte[] audioData) {
    AsrSession session = sessions.get(sessionId);
    if (!session.awaitReady(1200)) {
        throw new IllegalStateException("ASR session not ready");
    }
    String audioBase64 = Base64.getEncoder().encodeToString(audioData);
    session.getConversation().appendAudio(audioBase64);
}
```

Bước này rất quan trọng. `new OmniRealtimeConversation(...)` chỉ tạo object, còn `connect()` mới thiết lập connection; sau khi `updateSession(config)` gửi cấu hình, vẫn phải đợi event `session.updated` từ server. Trước khi nhận `asr_ready` từ backend, frontend nên disable microphone; nếu timeout ready hoặc nhận `error`, backend đóng connection cũ, thiết lập lại session và push state reconnect cho frontend. Tên callback ở trên chỉ mang tính minh họa, event type và method name thực tế phải được implement theo version DashScope SDK mà dự án sử dụng.

Khi server trả về kết quả nhận dạng, Handler sẽ push text incremental tới frontend:

```java
// Push text incremental qua WebSocket
websocket.sendMessage(new WebSocketSubtitleMessage(
    "subtitle",
    transcript,
    isFinal  // true nghĩa đây là kết quả cuối cùng
));
```

### Backend: service TTS

```java
// QwenTtsService.java
public byte[] synthesize(String text) throws Exception {
    CountDownLatch latch = new CountDownLatch(1);
    ByteArrayContainer audioContainer = new ByteArrayContainer();

    QwenTtsRealtime qwenTts = new QwenTtsRealtime(param, callback);
    try {
        qwenTts.connect();

        QwenTtsRealtimeConfig config = QwenTtsRealtimeConfig.builder()
            .voice(voice)  // Ví dụ "Cherry"
            .responseFormat(QwenTtsRealtimeAudioFormat.PCM_24000HZ_MONO_16BIT)
            .speechRate(speechRate)
            .build();

        qwenTts.updateSession(config);
        qwenTts.appendText(text);
        qwenTts.commit();

        if (!latch.await(30, TimeUnit.SECONDS)) {
            throw new TimeoutException("TTS synthesis timed out");
        }
        return audioContainer.toByteArray();
    } finally {
        qwenTts.close();
    }
}
```

Ví dụ đã lược bỏ audio callback và business exception mapping, nhưng không coi timeout là thành công. Implementation thực tế còn phải đóng connection khi cancel, đồng thời khôi phục interrupt flag của thread sau khi bắt `InterruptedException`.

Sau khi nhận PCM data, Handler chuyển thành WAV rồi push tới frontend:

```java
// Mỗi khi LLM output một câu hoàn chỉnh, submit vào TTS queue concurrent
OrderedTtsChunkEmitter chunkEmitter = new OrderedTtsChunkEmitter(session, semaphore);
llmService.chatStreamSentences(userText, sentence -> {
    chunkEmitter.submit(sentence);
});

// Push TTS chunk theo thứ tự câu, cuối cùng gửi control message audio_complete
chunkEmitter.finish();
chunkEmitter.awaitCompletion();
```

Điểm cần tối ưu ở đây là tổng thời gian chờ: **LLM vừa generate câu, TTS vừa tổng hợp, frontend vừa phát**. Backend dùng `max-concurrent-tts-per-session` để kiểm soát số lượng TTS concurrent trong một session, dùng `tts-timeout-seconds` để tránh một câu bị treo khiến cả lượt phát bị kẹt; nếu toàn bộ TTS cấp câu đều thất bại, fallback về tổng hợp toàn bộ text.

## Làm thế nào để Voice Agent hỗ trợ ngắt lời?

Ngắt lời là khó khăn thường gặp của Voice Agent, không thể giải quyết chỉ bằng một button pause.

### Ba tầng ý nghĩa của ngắt lời

1. **Ngắt ở playback layer**: khi người dùng nói, dừng audio hiện tại
2. **Ngắt ở generation layer**: cancel LLM và TTS đang generate ở server
3. **Ngắt ở context layer**: ghi chính xác nội dung đã phát và chưa phát

Version hiện tại của interview-guide vẫn chưa implement barge-in thực sự. Đoạn code dưới đây chỉ dừng gửi microphone audio tới backend trong khi AI phát, nhằm tránh echo trigger ASR; cái giá phải trả là lời người dùng nói chen cũng bị loại bỏ:

```typescript
// Implementation hiện tại: loại bỏ microphone audio trong khi AI phát
const handleAudioData = (audioData: string) => {
  if (isAiSpeakingRef.current) {
    return;
  }
  wsRef.current.sendAudio(audioData);
};

// Khi phát audio hoàn tất
const finishAiPlayback = () => {
  aiAudioPendingRef.current = false;
  clearAudioPlaybackWatchdog();
  setAiSpeaking(false);
  setIsSubmitting(false);

  // Sau khi phát bình thường hoàn tất, submit toàn bộ text
  commitAiMessage(aiTextRef.current.trim());
};
```

Để hỗ trợ ngắt lời thực sự, VAD phía edge phải tiếp tục phát hiện giọng người dùng trong lúc phát, sau đó dừng player hiện tại, xóa queue chưa phát và truyền `cancel` tới LLM/TTS đang chạy ở backend. Context cũng phải ghi lại text đã phát dựa trên tiến độ playback; `finishAiPlayback()` hiện tại chỉ submit toàn bộ nội dung sau khi phát hoàn tất, không có khả năng này.

### Ngắt lời dưới góc nhìn state machine

Xét từ góc độ state machine, ngắt lời là một control event có thể đi vào từ gần như mọi state:

| State hiện tại | Người dùng ngắt lời         | Response đúng                                                                                                                  |
| -------------- | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| listening      | Người dùng tiếp tục nói     | Tiếp tục thu âm và transcription; đây không phải ngắt lời                                                                      |
| thinking       | Người dùng bổ sung          | Cancel inference hiện tại, trigger lại bằng input mới                                                                          |
| speaking       | Người dùng nói chen         | Dừng playback, xóa queue                                                                                                       |
| tool_calling   | Người dùng nói “thôi bỏ đi” | Hủy ngay query có thể cancel; operation đã có side effect chuyển vào quy trình idempotent, compensation hoặc xác nhận thủ công |

Nếu hệ thống không có semantics cancel rõ ràng, rất nhanh sẽ xuất hiện trải nghiệm hỗn loạn kiểu “AI vừa nghe câu hỏi mới vừa phát câu trả lời cũ”.

## Browser audio capture và preprocessing đóng vai trò gì trong hệ thống giọng nói?

WebRTC thường được dùng một cách khái quát để chỉ capability audio video của browser. Voice Agent cần phân biệt API capture/preprocessing audio của browser với protocol truyền realtime WebRTC.

**Phân biệt quan trọng**:

- **Media Capture and Streams API** (`getUserMedia`): phụ trách thu audio từ microphone, có thể truyền constraint như AEC/NS/AGC và sample rate. Đây là thứ interview-guide thực sự dùng.
- **WebRTC protocol** (`RTCPeerConnection`): phụ trách truyền realtime end-to-end, bao gồm các protocol ICE, DTLS-SRTP, RTP. Chỉ khi kết nối mode WebRTC của OpenAI Realtime API, Azure Voice Live hoặc tự xây dựng chuỗi audio video realtime mới dùng đến transport layer này.

Đường đi audio của interview-guide là:

```
getUserMedia → AudioWorklet → Encode Base64 → Gửi qua WebSocket
```

Transport layer của đường đi này là **WebSocket (TCP)**, không phải **RTP/SRTP** của WebRTC. WebSocket đảm bảo thứ tự, nhưng trong mạng yếu sẽ chịu ảnh hưởng của TCP retransmission; WebRTC thường ưu tiên UDP, kết hợp jitter buffer, packet loss concealment và các cơ chế khác để giảm giật audio realtime, khi mạng bị hạn chế cũng có thể fallback sang TCP/TURN.

### Pipeline preprocessing audio của browser

Trong trường hợp sử dụng Voice Agent, các capability preprocessing audio của browser thường dùng là:

```
Input microphone
    │
    ▼
┌─────────────────────────┐
│  AEC (khử echo)         │  Khử âm thanh do loa phát
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│  NS (khử noise)         │  Giảm background noise
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│  AGC (điều chỉnh gain)  │  Giúp âm lượng ổn định hơn
└─────────────────────────┘
    │
    ▼
┌─────────────────────────┐
│  VAD (phát hiện voice)  │  Xác định có giọng người hay không
└─────────────────────────┘
    │
    ▼
Output sau encode
```

### Lựa chọn cấu hình getUserMedia

interview-guide dùng cấu hình `getUserMedia` cơ bản nhất:

```typescript
navigator.mediaDevices.getUserMedia({
  audio: {
    echoCancellation: true,
    noiseSuppression: true,
    autoGainControl: true,
    sampleRate: 16000,
  },
});
```

Nhưng đây không phải lựa chọn duy nhất, mỗi trường hợp có sự đánh đổi khác nhau:

| Parameter        | true                                                             | false                                          | Gợi ý                                                                            |
| ---------------- | ---------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------- |
| echoCancellation | Khử echo loa nhưng làm mất một phần chất lượng audio             | Giữ chất lượng audio gốc nhưng phải tự làm AEC | Bật                                                                              |
| noiseSuppression | Giảm noise nhưng có thể làm mất cả giọng người dùng              | Phải tự làm NS                                 | Bật khi môi trường ồn, tắt khi yên tĩnh                                          |
| autoGainControl  | Tự động điều chỉnh âm lượng về khoảng phù hợp                    | Phụ thuộc âm lượng gốc của microphone          | Bật                                                                              |
| sampleRate       | Càng cao thì chất lượng audio càng tốt nhưng lượng data càng lớn | 16 kHz đã đủ với phần lớn ASR                  | Cấu hình theo yêu cầu của model; browser không nhất thiết output đúng constraint |

Hiệu quả của AEC/NS/AGC khác nhau khá nhiều giữa các browser và thiết bị. Cần test riêng desktop Chrome, Safari và mobile; trường hợp test ít nhất phải bao gồm loa ngoài, tai nghe, phòng họp và mạng mobile.

### Ranh giới của WebRTC

WebRTC rất phù hợp với audio realtime trên browser, nhưng nếu xây dựng phương án App hoặc hardware thì phải xem xét capability của platform và ràng buộc về power consumption.

Phát triển native trên mobile có thể dùng:

- **iOS**: AVAudioEngine + audio processing tích hợp sẵn của hệ thống
- **Android**: AudioRecord + Oboe/AAudio, hoặc thư viện WebRTC của Google

Trường hợp hardware (smart speaker, thiết bị trên xe) thường cần DSP chuyên dụng hoặc algorithm audio frontend để xử lý echo, beamforming của microphone array và thu âm ở khoảng cách xa; chỉ preprocessing phần mềm kiểu browser là không đủ.

## Cascade chain và native realtime model có ưu nhược điểm gì?

Đây là vấn đề cốt lõi khi lựa chọn.

### Phương án một: Cascade ASR + LLM + TTS

```
Audio -> VAD -> Streaming ASR -> LLM -> Streaming TTS -> Audio
```

Ưu điểm:

- Text ASR có thể lưu DB, audit và sửa lỗi
- Input/output của LLM đều là text, thuận tiện reuse Agent framework hiện có
- Có thể thay voice và vendor của TTS độc lập
- Có thể benchmark và tối ưu từng component riêng

Nhược điểm:

- Mỗi layer đều có latency
- Lỗi ASR truyền sang LLM
- Layer trung gian text làm mất ngữ điệu, khoảng dừng và cảm xúc
- Ngắt lời phải cancel thống nhất xuyên suốt ASR, LLM, TTS và player

interview-guide chính là phương án này. Các trường hợp phù hợp: hỏi đáp tri thức doanh nghiệp, ticket CSKH và hệ thống nghiệp vụ cần audit compliance.

### Phương án hai: Native Realtime Speech-to-Speech

```
Audio -> Native multimodal model -> Audio
```

Phương án tiêu biểu: OpenAI Realtime API, Gemini Live API, Alibaba Qwen-Omni.

Ưu điểm:

- Latency end-to-end thấp hơn
- Giữ lại nhiều hơn các thông tin phi ngôn ngữ như ngữ điệu, khoảng dừng và cảm xúc
- Có thể thống nhất xử lý audio input, text event và tool calling

Nhược điểm:

- Quá trình trung gian black box hơn, việc định vị vấn đề phụ thuộc nhiều hơn vào log của vendor
- Audit text và kiểm soát thoại cần thiết kế thêm
- Mô hình chi phí có thể tính theo audio token hoặc thời lượng
- Nếu nghiệp vụ phụ thuộc nhiều vào private deployment, API của vendor có thể không đáp ứng yêu cầu

**Lựa chọn cách kết nối**:

Tài liệu hiện tại của OpenAI Realtime API cung cấp ba cách kết nối:

| Cách kết nối | Trường hợp sử dụng                                                                  |
| ------------ | ----------------------------------------------------------------------------------- |
| WebRTC       | Browser và ứng dụng mobile, phù hợp để thu microphone và phát audio model trực tiếp |
| WebSocket    | Trường hợp middleware server-to-server, latency thấp và dễ kiểm soát                |
| SIP          | Tích hợp hệ thống điện thoại VoIP, phù hợp với call center và CSKH qua điện thoại   |

### Gợi ý của tôi

Với sản phẩm voice có tần suất cao, realtime mạnh và nhấn mạnh tương tác tự nhiên, có thể ưu tiên đánh giá Native Realtime API. Khi cần audit text ASR từng bước, kiểm soát thoại response hoặc private deployment, cascade chain thường dễ quan sát và thay component hơn, nhưng việc tăng số component cũng đưa vào nhiều điểm timeout và failure hơn.

**Đừng làm hybrid edge-cloud ngay từ ngày đầu**. Trước tiên hãy chạy thông suốt một chain, sau đó thay thế từng bước.

## Tối ưu hệ thống giọng nói trong môi trường production như thế nào?

### 1. Điều chỉnh audio frame và uplink chunk

Encode và xử lý giọng nói thường dùng frame length như 10 ms, 20 ms, 30 ms; ở application layer có thể gộp nhiều frame thành một network chunk. Frame length và uplink chunk là hai khái niệm khác nhau: processing frame quá lớn sẽ làm tăng algorithm latency, còn network chunk quá nhỏ sẽ làm tăng message và encode overhead.

Lựa chọn của interview-guide là **chunk 200 ms**:

```typescript
// pcm-processor.js
this.targetSampleRate = 16000;
this.samplesPerChunk = 3200; // 200 ms ở 16 kHz
```

Điều này không khiến ASR phải đợi hết cả câu mới bắt đầu làm việc, nhưng sẽ đưa vào audio uplink tối đa một chu kỳ chunk; cộng thêm thời gian ngắt câu im lặng của VAD phía server, người dùng sẽ cảm thấy “sau khi dứt lời vẫn phải đợi một chút”. Nếu muốn làm tốt hơn, có thể:

- Giảm chunk xuống 100 ms
- Frontend gửi trước một đoạn ngắn để ASR “warm start”
- Dùng transcription incremental ổn định của ASR để phán đoán intent sớm; event VAD chỉ phụ trách cung cấp hoạt động giọng nói và ranh giới lượt

### 2. Để LLM nói câu ngắn trước

Response bằng giọng nói không phải bài viết. Người dùng không cần vừa bắt đầu đã nghe một câu trả lời hoàn chỉnh 500 chữ.

Strategy tốt hơn:

- Trước tiên output câu xác nhận: “để tôi xem”
- Trong lúc tool calling, phát câu chuyển tiếp: “đang tra đơn hàng gần nhất”
- Sau khi có kết quả mới đưa ra kết luận
- Chia phần giải thích dài thành nhiều câu, mỗi câu đều có thể tổng hợp độc lập

### 3. Tách TTS theo ranh giới ngữ nghĩa

Tách TTS quá vụn sẽ khiến âm thanh đứt quãng; tách quá dài làm latency audio đầu tiên cao.

Khuyến nghị tách theo thứ tự ưu tiên:

1. Dấu chấm, dấu hỏi, dấu chấm than
2. Dấu chấm phẩy, dấu hai chấm
3. Cụm ngắn dài hơn có dấu phẩy
4. Cưỡng chế tách câu quá dài

Đồng thời cần tránh tách hỏng số, viết tắt tiếng Anh và tên code. Ví dụ "GPT-4o-mini-tts" không thể tùy tiện tách thành vài đoạn để đọc.

interview-guide hiện áp dụng đúng cách này: trong quá trình LLM output streaming, chỉ cần phát hiện một câu hoàn chỉnh là lập tức submit cho `OrderedTtsChunkEmitter` để TTS cấp câu. Frontend nhận `audio_chunk` thì lập tức đưa vào queue để phát, nhận `audio_complete` rồi mới đợi playback queue tự nhiên rỗng. Nhờ vậy audio đầu tiên không cần đợi toàn bộ response được generate và tổng hợp xong.

### 4. Kiểm soát độ dài context

Voice Agent rất dễ đưa toàn bộ transcription, kết quả tool và playback state vào context. Ngắn hạn có thể không sao, nhưng trong session dài latency và chi phí sẽ cùng tăng.

Khuyến nghị chia context thành ba tầng:

- **Raw data ngắn hạn**: transcription và response đầy đủ của vài lượt gần nhất
- **Session summary**: mục tiêu của người dùng, fact đã xác nhận và việc chưa hoàn thành
- **Event state**: tiến độ playback hiện tại, có bị ngắt hay không và kết quả tool calling

LLM không cần biết từng audio frame đã xảy ra gì, nó cần biết state có signal-to-noise ratio cao và liên quan đến quyết định hiện tại.

### 5. Observability end-to-end

interview-guide dùng Redis để cache session state:

```java
// VoiceInterviewService.java
private static final String SESSION_CACHE_KEY_PREFIX = "voice:interview:session:";

private void cacheSession(VoiceInterviewSessionEntity session) {
    String cacheKey = getSessionCacheKey(session.getId());
    RBucket<VoiceInterviewSessionEntity> bucket = redissonClient.getBucket(cacheKey);
    bucket.set(session, Duration.ofHours(CACHE_TTL_HOURS));
}
```

Production còn phải ghi lại:

- Thời lượng audio uplink
- Thời lượng voice hữu hiệu
- ASR token hoặc số phút
- Input/output token của LLM
- Số ký tự TTS, số giây audio và số giây bị ngắt
- Latency end-to-end của mỗi lượt và số lần cancel

Không có những metric này, chi phí của Voice Agent sẽ rất khó hội tụ.

## Voice Agent còn có thể phát triển như thế nào?

interview-guide là version cơ bản nhất, còn rất nhiều điểm có thể tối ưu.

### Hybrid edge-cloud

Hiện tại interview-guide về cơ bản là thiết kế “cloud là chính”. Hướng nâng cao là đưa thêm capability xuống edge:

| Khâu | Hiện tại                        | Hướng phát triển                                               |
| ---- | ------------------------------- | -------------------------------------------------------------- |
| VAD  | VAD phía edge + VAD phía server | VAD hoàn toàn phía edge, giảm tải server                       |
| ASR  | Hoàn toàn trên cloud            | Command đơn giản đặt ở edge, nhận dạng phức tạp đặt trên cloud |
| LLM  | Hoàn toàn trên cloud            | Model nhỏ làm fallback phía edge, vẫn dùng được khi mất mạng   |
| TTS  | Hoàn toàn trên cloud            | Prompt cố định đặt ở edge, hội thoại tự nhiên đặt trên cloud   |

Hybrid edge-cloud không có nghĩa là nhét tất cả model vào client. Cách ổn định hơn là: ưu tiên đưa xuống edge những capability yêu cầu realtime mạnh, nhạy cảm về privacy hoặc cần fallback khi mất mạng; giữ trên cloud những capability cần model lớn hiểu, suy luận phức tạp và audit thống nhất.

### Deploy model local

Nếu có yêu cầu về compliance dữ liệu, có thể cân nhắc deploy ASR và TTS local:

- **ASR**: faster-whisper, FunASR, SenseVoice
- **TTS**: piper1-gpl (Piper gốc đã bị archive), Fish Speech, CosyVoice

**Lưu ý**: repository Piper gốc (rhasspy/piper) đã được archive vào tháng 10 năm 2025, việc phát triển đã chuyển sang [OHF-Voice/piper1-gpl](https://github.com/OHF-Voice/piper1-gpl). piper1-gpl dùng GPL-3.0, project commercial cần đánh giá yêu cầu compliance với open source; project hiện cũng đang tuyển maintainer mới, nên support dài hạn còn chưa chắc chắn. Fish Speech không dùng Apache 2.0: model, code và tài liệu hiện tại của nó chịu ràng buộc bởi [Fish Audio Research License](https://github.com/fishaudio/fish-speech/blob/main/LICENSE); nghiên cứu và mục đích non-commercial có thể dùng miễn phí, còn mục đích commercial cần xin giấy phép bằng văn bản riêng từ Fish Audio. CosyVoice cũng cần kiểm tra riêng license của code, model weights và voice resource.

Ưu điểm của deploy local là dễ kiểm soát và có thể offline. Nhược điểm là **chi phí engineering cao**: phải tự benchmark GPU, memory và capacity concurrent, đồng thời tự xử lý inference streaming, hot loading model và thu hồi VRAM.

### Native Realtime API

Nếu latency và độ tự nhiên của cascade chain đã không thể giảm thêm, có thể đánh giá Native Realtime API:

- OpenAI Realtime API (hỗ trợ WebRTC, WebSocket và SIP, tên model cụ thể được đọc từ config hoặc model gateway)
- Gemini Live API
- Alibaba Qwen-Omni

Các API này hợp nhất ASR, LLM và TTS thành một chain multimodal thống nhất, thường có ưu thế về latency và độ tự nhiên. Cái giá cũng rất thực tế: quá trình trung gian black box hơn, mô hình chi phí thay đổi nhanh, việc debug và audit đều cần thiết kế thêm.

Vào tháng 8 năm 2025, OpenAI đưa Realtime API lên GA và phát hành voice model chuyên dụng `gpt-realtime`. Realtime model được update khá nhanh; code production không nên rải model name trong business logic, mà nên do config center hoặc model gateway quản lý thống nhất, đồng thời kiểm tra model catalog hiện tại trước khi release.

Khi phát hành GA, Realtime API đồng thời cung cấp hoặc bổ sung một số capability:

1. **Hỗ trợ remote MCP server**, có thể gọi external tool như cascade chain;
2. **Hỗ trợ image input**, model có thể kết hợp nội dung màn hình người dùng đang nhìn để hội thoại;
3. **Tích hợp điện thoại SIP**, hỗ trợ kết nối với mạng điện thoại truyền thống.

Cũng không nên hard-code price. Realtime model thường phân biệt cách tính phí cho text, audio, cached input và output; trước khi tích hợp thực tế, nhất định phải lấy trang pricing chính thức làm chuẩn.

### Tối ưu trải nghiệm ngắt lời

Hiện tại cách ngắt lời của interview-guide là “âm thầm loại bỏ”: âm thanh của người dùng không được gửi đi khi AI đang nói. Cách này đơn giản nhưng trải nghiệm chưa tự nhiên.

Cách tốt hơn:

- Khi AI nói, tiếp tục nhận audio nhưng không gửi tới ASR
- Sau khi phát hiện giọng người dùng, trước tiên giảm âm lượng playback của AI (fade thay vì dừng đột ngột)
- Sau khi ngắt lời, giữ context của phần đã phát

### Mở rộng multimodal

interview-guide hiện chỉ có voice. Có thể mở rộng thành:

- **Voice + chia sẻ màn hình**: interviewer có thể nhìn thấy IDE của ứng viên
- **Voice + camera**: xem biểu cảm và ngôn ngữ cơ thể của ứng viên
- **Voice + whiteboard**: cùng vẽ architecture diagram

Các capability multimodal này cần flow management và state synchronization phức tạp hơn.

## Trả lời câu hỏi về hệ thống giọng nói AI trong phỏng vấn như thế nào?

Nếu interviewer hỏi: “Bạn thiết kế một Voice Agent realtime như thế nào?”

Có thể trả lời theo cách này:

1. **Trước tiên tách chain**: client thu audio, VAD xác định ranh giới lời nói, ASR transcription streaming, LLM thực hiện hiểu intent và tool calling, TTS tổng hợp streaming, client vừa nhận vừa phát.
2. **Sau đó nói về khó khăn**: khó khăn cốt lõi của voice realtime là latency end-to-end, ngắt lời người dùng, môi trường nhiễu, context state và phối hợp edge-cloud.
3. **Tiếp theo nói về state machine**: cần quản lý các state listening, thinking, speaking, interrupted..., khi ngắt lời phải cancel playback, cancel generation và xử lý context đã phát/chưa phát.
4. **Cuối cùng nói về lựa chọn**: API cloud lên production nhanh, model local dễ kiểm soát nhưng chi phí engineering cao; hybrid edge-cloud và Speech-to-Speech API có phù hợp hay không phải đánh giá theo latency, compliance, chi phí và nhu cầu observability.

## Tổng kết

Voice Agent AI là một chain audio realtime: VAD quyết định lượt hội thoại, ASR, LLM và TTS cần nối streaming, playback, ngắt lời, cancel và session state phải được xử lý phối hợp. Cascade chain dễ kiểm soát và audit quá trình trung gian hơn, native realtime model có thể có ưu thế hơn về latency và độ tự nhiên; còn việc phân chia edge-cloud phải đánh giá kết hợp realtime, privacy, fallback offline, chi phí và năng lực vận hành. Dù dùng phương án nào, cũng nên thiết kế nó thành một hệ thống có thể cancel, observable và degrade, thay vì chỉ nối đơn giản ba lần gọi API.
