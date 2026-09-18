---
title: Một nghìn dòng ghi chú học MySQL
description: Tổng hợp tinh hoa ghi chú học MySQL trong một nghìn dòng, bao quát thao tác database, quản lý bảng, cú pháp SQL, index, view, stored procedure, trigger và các kiến thức cốt lõi khác, phù hợp để tra cứu và ôn tập nhanh.
category: Database
tag:
  - MySQL
head:
  - - meta
    - name: keywords
      content: Ghi chú học MySQL, tổng hợp lệnh MySQL, cú pháp SQL, thao tác database, thao tác bảng, index, view, stored procedure, trigger
---

> Địa chỉ bản gốc: <https://shockerli.net/post/1000-line-mysql-note/> , JavaGuide đã biên tập lại bài viết và bổ sung mục lục.

Một bản tổng hợp rất hay, đặc biệt khuyến nghị bạn lưu lại để xem khi cần.

### Thao tác cơ bản

```sql
/* Windows service */
-- Khởi động MySQL
            net start mysql
-- Tạo Windows service
                sc create mysql binPath= mysqld_bin_path (lưu ý: có khoảng trắng giữa dấu bằng và giá trị)
/* Kết nối và ngắt kết nối server */
-- Kết nối MySQL
                mysql -h address -P port -u username -p password
-- Hiển thị các thread đang chạy
                SHOW PROCESSLIST
-- Hiển thị thông tin system variable
                SHOW VARIABLES
```

### Thao tác database

```sql
/* Thao tác database */
-- Xem database hiện tại
    SELECT DATABASE();
-- Hiển thị thời gian hiện tại, username, phiên bản database
    SELECT now(), user(), version();
-- Tạo database
    CREATE DATABASE[ IF NOT EXISTS] database_name database_options
    database_options:
        CHARACTER SET charset_name
        COLLATE collation_name
-- Xem các database hiện có
    SHOW DATABASES[ LIKE 'PATTERN']
-- Xem thông tin database hiện tại
    SHOW CREATE DATABASE database_name
-- Sửa thông tin option của database
    ALTER DATABASE database_name option_info
-- Xóa database
    DROP DATABASE[ IF EXISTS] database_name
        Đồng thời xóa directory liên quan và nội dung của directory đó
```

### Thao tác bảng

```sql
/* Thao tác bảng */
-- Tạo bảng
    CREATE [TEMPORARY] TABLE[ IF NOT EXISTS] [database_name.]table_name ( table_definition )[ table_options]
        Mỗi field phải có data type
        Sau field cuối cùng không được có dấu phẩy
        TEMPORARY là temporary table, table tự động biến mất khi session kết thúc
        Định nghĩa field:
            field_name data_type [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
-- Table options
    -- Character set
        CHARSET = charset_name
        Nếu table không được thiết lập thì dùng character set của database
    -- Storage engine
        ENGINE = engine_name
        Các data structure khác nhau được dùng khi table quản lý dữ liệu; structure khác nhau sẽ dẫn đến cách xử lý, tính năng và thao tác được cung cấp khác nhau
        Các engine thường gặp: InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
        Các engine khác nhau dùng cách khác nhau để lưu structure và dữ liệu của table
        Ý nghĩa file của MyISAM: .frm là table definition, .MYD là table data, .MYI là index
        Ý nghĩa file của InnoDB: .frm là table definition, table space data và log file
        SHOW ENGINES -- Hiển thị thông tin trạng thái của storage engine
        SHOW ENGINE engine_name {LOGS|STATUS} -- Hiển thị log hoặc thông tin trạng thái của storage engine
    -- Giá trị bắt đầu tự tăng
        AUTO_INCREMENT = row_count
    -- Directory data file
        DATA DIRECTORY = 'directory'
    -- Directory index file
        INDEX DIRECTORY = 'directory'
    -- Comment của table
        COMMENT = 'string'
    -- Partition option
        PARTITION BY ... (xem chi tiết trong manual)
-- Xem tất cả table
    SHOW TABLES[ LIKE 'pattern']
    SHOW TABLES FROM database_name
-- Xem table structure
    SHOW CREATE TABLE table_name (thông tin chi tiết hơn)
    DESC table_name / DESCRIBE table_name / EXPLAIN table_name / SHOW COLUMNS FROM table_name [LIKE 'PATTERN']
    SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']
-- Sửa table
    -- Sửa option của chính table
        ALTER TABLE table_name table_options
        eg: ALTER TABLE table_name ENGINE=MYISAM;
    -- Đổi tên table
        RENAME TABLE old_table_name TO new_table_name
        RENAME TABLE old_table_name TO database_name.table_name (có thể chuyển table sang database khác)
        -- RENAME có thể hoán đổi hai table name
    -- Sửa cấu trúc field của table (13.1.2. Cú pháp ALTER TABLE)
        ALTER TABLE table_name operation
        -- operation
            ADD[ COLUMN] field_definition       -- Thêm field
                AFTER field_name          -- Thêm sau field này
                FIRST                     -- Thêm vào đầu tiên
            ADD PRIMARY KEY(field_name)   -- Tạo primary key
            ADD UNIQUE [index_name] (field_name) -- Tạo unique index
            ADD INDEX [index_name] (field_name) -- Tạo index thông thường
            DROP[ COLUMN] field_name      -- Xóa field
            MODIFY[ COLUMN] field_name field_attributes     -- Hỗ trợ sửa field attributes, không thể sửa field name (cũng phải ghi lại mọi attribute cũ)
            CHANGE[ COLUMN] old_field_name new_field_name field_attributes      -- Hỗ trợ sửa field name
            DROP PRIMARY KEY    -- Xóa primary key (phải xóa attribute AUTO_INCREMENT trước khi xóa primary key)
            DROP INDEX index_name -- Xóa index
            DROP FOREIGN KEY foreign_key -- Xóa foreign key
-- Xóa table
    DROP TABLE[ IF EXISTS] table_name ...
-- Xóa dữ liệu trong table
    TRUNCATE [TABLE] table_name
-- Sao chép table structure
    CREATE TABLE table_name LIKE source_table_name
-- Sao chép table structure và data
    CREATE TABLE table_name [AS] SELECT * FROM source_table_name
-- Kiểm tra table có lỗi hay không
    CHECK TABLE tbl_name [, tbl_name] ... [option] ...
-- Tối ưu table
    OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
-- Sửa table
    REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]
-- Phân tích table
    ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

### Thao tác dữ liệu

```sql
/* Thao tác dữ liệu */ ------------------
-- Insert
    INSERT [INTO] table_name [(field_list)] VALUES (value_list)[, (value_list), ...]
        -- Nếu value list cần insert chứa tất cả field và theo đúng thứ tự thì có thể bỏ qua field list.
        -- Có thể insert nhiều data record cùng lúc!
        REPLACE tương tự INSERT. Điểm khác biệt duy nhất là với row khớp (so sánh data của row hiện có với primary key/unique key), row hiện có sẽ bị thay thế; nếu chưa có row thì insert row mới.
    INSERT [INTO] table_name SET field_name=value[, field_name=value, ...]
-- Select
    SELECT field_list FROM table_name[ other_clauses]
        -- Có thể lấy nhiều field từ nhiều table
        -- Có thể không dùng other clauses
        -- Có thể thay field list bằng *, biểu thị tất cả field
-- Delete
    DELETE FROM table_name[ delete_condition_clause]
        Không có condition clause thì sẽ xóa toàn bộ
-- Update
    UPDATE table_name SET field_name=new_value[, field_name=new_value] [update_condition]
```

### Character set encoding

```sql
/* Character set encoding */ ------------------
-- MySQL, database, table và field đều có thể thiết lập encoding
-- Data encoding không cần giống client encoding
SHOW VARIABLES LIKE 'character_set_%'   -- Xem tất cả mục character set encoding
    character_set_client        Encoding dùng khi client gửi data đến server
    character_set_results       Encoding server dùng để trả result về client
    character_set_connection    Encoding của connection layer
SET variable_name = variable_value
    SET character_set_client = gbk;
    SET character_set_results = gbk;
    SET character_set_connection = gbk;
SET NAMES GBK;  -- Tương đương hoàn tất ba thiết lập trên
-- Collation
    Collation dùng để sort
    SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern']   Xem tất cả character set
    SHOW COLLATION [LIKE 'pattern']     Xem tất cả collation
    CHARSET character_set_encoding     Thiết lập character set encoding
    COLLATE collation_encoding          Thiết lập collation encoding
```

### Data type (column type)

```sql
/* Data type (column type) */ ------------------
1. Numeric type
-- a. Integer ----------
    Type         Byte     Range (signed)
    tinyint     1 byte    -128 ~ 127      Unsigned: 0 ~ 255
    smallint    2 byte    -32768 ~ 32767
    mediumint   3 byte    -8388608 ~ 8388607
    int         4 byte
    bigint      8 byte
    int(M)  M biểu thị tổng số chữ số
    - Mặc định có sign bit, có thể sửa bằng attribute unsigned
    - Display width: nếu một số không đủ số chữ số được định nghĩa cho field thì bổ sung 0 ở trước, có thể sửa bằng attribute zerofill
        Ví dụ: int(5)   insert số '123', sau khi bổ sung là '00123'
    - Nếu đáp ứng yêu cầu thì nên chọn loại nhỏ hơn.
    - 1 biểu thị giá trị bool true, 0 biểu thị giá trị bool false. MySQL không có boolean type, dùng integer 0 và 1 để biểu thị. Thường dùng tinyint(1) để biểu thị boolean.
-- b. Floating-point type ----------
    Type             Byte     Range
    float (single precision)     4 byte
    double (double precision)    8 byte
    Floating-point type hỗ trợ cả sign bit, attribute unsigned và display width, attribute zerofill.
        Khác integer type, cả trước và sau đều được bổ sung 0.
    Khi định nghĩa floating-point type cần chỉ định tổng số chữ số và số chữ số thập phân.
        float(M, D)     double(M, D)
        M biểu thị tổng số chữ số, D biểu thị số chữ số thập phân.
        Kích thước của M và D quyết định range của floating-point number. Khác với range cố định của integer type.
        M vừa biểu thị tổng số chữ số (không gồm dấu chấm thập phân và dấu dương/âm), vừa biểu thị display width (gồm tất cả display symbol).
        Hỗ trợ biểu diễn bằng scientific notation.
        Floating-point number biểu thị giá trị gần đúng.
-- c. Fixed-point number ----------
    decimal -- Độ dài thay đổi
    decimal(M, D)   M cũng biểu thị tổng số chữ số, D biểu thị số chữ số thập phân.
    Lưu giá trị chính xác, không xảy ra thay đổi dữ liệu do làm tròn như floating-point number.
    Chuyển floating-point number thành string để lưu, cứ 9 chữ số được lưu bằng 4 byte.
2. String type
-- a. char, varchar ----------
    char    Fixed-length string, tốc độ nhanh nhưng lãng phí không gian
    varchar  Variable-length string, tốc độ chậm nhưng tiết kiệm không gian
    M biểu thị độ dài tối đa có thể lưu, độ dài này tính theo số character, không phải số byte.
    Encoding khác nhau chiếm không gian khác nhau.
    char tối đa 255 character, không phụ thuộc encoding.
    varchar tối đa 65535 character, phụ thuộc encoding.
    Một record hợp lệ không được vượt quá 65535 byte.
        utf8 tối đa 21844 character, gbk tối đa 32766 character, latin1 tối đa 65532 character
    varchar có độ dài thay đổi nên cần dùng storage space để lưu độ dài varchar. Nếu data nhỏ hơn 255 byte thì dùng 1 byte để lưu độ dài, ngược lại cần 2 byte.
    Độ dài hợp lệ tối đa của varchar được xác định bởi row size tối đa và character set được sử dụng.
    Độ dài hợp lệ tối đa là 65532 byte, vì khi lưu string trong varchar, byte đầu tiên là byte trống, không chứa data, sau đó cần thêm 2 byte để lưu độ dài string, nên độ dài hợp lệ là 65535-1-2=65532 byte.
    Ví dụ: nếu một table được định nghĩa là CREATE TABLE tb(c1 int, c2 char(30), c3 varchar(N)) charset=utf8; N lớn nhất là bao nhiêu? Đáp án: (65535-1-2-4-30*3)/3
-- b. blob, text ----------
    blob Binary string (byte string)
        tinyblob, blob, mediumblob, longblob
    text Non-binary string (character string)
        tinytext, text, mediumtext, longtext
    Khi định nghĩa text không cần định nghĩa length và cũng không tính tổng length.
    Khi định nghĩa text type không thể đặt default value
-- c. binary, varbinary ----------
    Tương tự char và varchar, dùng để lưu binary string, tức byte string thay vì character string.
    char, varchar, text tương ứng với binary, varbinary, blob.
3. Date and time type
    Thông thường dùng integer để lưu timestamp vì PHP có thể format timestamp rất thuận tiện.
    datetime    8 byte    Date and time     1000-01-01 00:00:00 đến 9999-12-31 23:59:59
    date        3 byte    Date              1000-01-01 đến 9999-12-31
    timestamp   4 byte    Timestamp         19700101000000 đến 2038-01-19 03:14:07
    time        3 byte    Time              -838:59:59 đến 838:59:59
    year        1 byte    Year              1901 - 2155
    datetime    YYYY-MM-DD hh:mm:ss
    timestamp   YY-MM-DD hh:mm:ss
                YYYYMMDDhhmmss
                YYMMDDhhmmss
                YYYYMMDDhhmmss
                YYMMDDhhmmss
    date        YYYY-MM-DD
                YY-MM-DD
                YYYYMMDD
                YYMMDD
                YYYYMMDD
                YYMMDD
    time        hh:mm:ss
                hhmmss
                hhmmss
    year        YYYY
                YY
                YYYY
                YY
4. Enumeration and set
-- enum ----------
enum(val1, val2, val3...)
    Chọn một giá trị trong các giá trị đã biết. Số lượng tối đa là 65535.
    Khi lưu, enum value được lưu dưới dạng integer 2 byte (smallint). Mỗi enum value tăng dần từ 1 theo thứ tự vị trí lưu.
    Hiển thị dưới dạng string nhưng lưu dưới dạng integer.
    Index của NULL value là NULL.
    Index của empty string error value là 0.
-- set ----------
set(val1, val2, val3...)
    create table tab ( gender set('male', 'female', 'none') );
    insert into tab values ('male, female');
    Tối đa có 64 member khác nhau. Lưu bằng bigint, chiếm tổng cộng 8 byte. Sử dụng dạng bit operation.
    Khi tạo table, trailing space của SET member value sẽ tự động bị xóa.
```

### Column attribute (column constraint)

```sql
/* Column attribute (column constraint) */ ------------------
1. PRIMARY key
    - Field có thể nhận diện record duy nhất có thể làm primary key.
    - Một table chỉ có một primary key.
    - Primary key có tính unique.
    - Khi khai báo field, dùng primary key để đánh dấu.
        Cũng có thể khai báo sau field list
            Ví dụ: create table tab ( id int, stu varchar(10), primary key (id));
    - Giá trị của primary key field không được là null.
    - Primary key có thể gồm nhiều field. Khi đó cần khai báo sau field list.
        Ví dụ: create table tab ( id int, stu varchar(10), age int, primary key (stu, age));
2. UNIQUE unique index (unique constraint)
    Khiến giá trị của field cũng không được trùng lặp.
3. NULL constraint
    null không phải data type mà là một attribute của column.
    Biểu thị column hiện tại có thể là null hay không, tức là không có gì.
    null, cho phép rỗng. Mặc định.
    not null, không cho phép rỗng.
    insert into tab values (null, 'val');
        -- Lúc này đặt giá trị của field đầu tiên thành null, tùy thuộc field đó có cho phép null hay không
4. DEFAULT default value attribute
    Default value của field hiện tại.
    insert into tab values (default, 'val');    -- Lúc này bắt buộc dùng default value.
    create table tab ( add_time timestamp default current_timestamp );
        -- Đặt timestamp của thời gian hiện tại làm default value.
        current_date, current_time
5. AUTO_INCREMENT auto-increment constraint
    Auto-increment phải là index (primary key hoặc unique)
    Chỉ có thể có một field auto-increment.
    Mặc định bắt đầu auto-increment từ 1. Có thể thiết lập bằng table attribute auto_increment = x hoặc alter table tbl auto_increment = x;
6. COMMENT comment
    Ví dụ: create table tab ( id int ) comment 'comment content';
7. FOREIGN KEY foreign key constraint
    Dùng để giới hạn data integrity giữa main table và child table.
    alter table t1 add constraint `t1_t2_fk` foreign key (t1_id) references t2(id);
        -- Liên kết foreign key t1_id của table t1 với field id của table t2.
        -- Mỗi foreign key có một name, có thể chỉ định bằng constraint
    Table có foreign key được gọi là child table, table mà foreign key trỏ đến được gọi là main table.
    Tác dụng: duy trì data consistency và integrity, mục đích chính là kiểm soát data được lưu trong foreign key table (child table).
    Trong MySQL, có thể dùng foreign key constraint với InnoDB engine:
    Syntax:
    foreign key (foreign_key_field) references main_table_name (referenced_field) [action on main record deletion] [action on main record update]
    Khi đó cần kiểm tra foreign key của child table phải ràng buộc với một value đã tồn tại trong main table. Khi chưa có liên kết, foreign key có thể đặt là null, với điều kiện foreign key column không có not null.
    Có thể không chỉ định action khi main record bị thay đổi hoặc update; khi đó thao tác của main table sẽ bị từ chối.
    Nếu chỉ định on update hoặc on delete thì khi delete hoặc update có thể chọn các thao tác sau:
    1. cascade, thao tác cascade. Khi data của main table được update (primary key value được update), child table cũng được update (foreign key value được update). Khi main record bị delete, các record liên quan trong child table cũng bị delete.
    2. set null, đặt thành null. Khi data của main table được update (primary key value được update), foreign key của child table được đặt thành null. Khi main record bị delete, foreign key của record liên quan trong child table được đặt thành null. Tuy nhiên foreign key column không được có constraint attribute not null.
    3. restrict, từ chối delete và update main table.
    Lưu ý, foreign key chỉ được InnoDB storage engine hỗ trợ. Engine khác không hỗ trợ.

```

### Quy chuẩn tạo table

```sql
/* Quy chuẩn tạo table */ ------------------
    -- Normal Format, NF
        - Mỗi table lưu thông tin của một entity
        - Mỗi table có một ID field làm primary key
        - ID primary key + atomic table
    -- 1NF, First Normal Form
        Field không thể tách nhỏ hơn thì thỏa mãn First Normal Form.
    -- 2NF, Second Normal Form
        Trên tiền đề thỏa mãn First Normal Form, không được xuất hiện partial dependency.
        Loại bỏ composite primary key có thể tránh partial dependency. Thêm single-column key.
    -- 3NF, Third Normal Form
        Trên tiền đề thỏa mãn Second Normal Form, không được xuất hiện transitive dependency.
        Một field phụ thuộc vào primary key, trong khi field khác phụ thuộc vào field đó. Đây là transitive dependency.
        Đặt data của một entity vào một table để triển khai.
```

### SELECT

```sql
/* SELECT */ ------------------
SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [aggregate function] -> HAVING -> ORDER BY -> LIMIT
a. select_expr
    -- Có thể dùng * để biểu thị tất cả field.
        select * from tb;
    -- Có thể dùng expression (công thức tính, function call, field cũng là một expression)
        select stu, 29+25, now() from tb;
    -- Có thể đặt alias cho từng column. Dùng để đơn giản hóa column identifier và tránh nhiều column identifier trùng nhau.
        - Dùng keyword as, cũng có thể bỏ qua as.
        select stu+10 as add10 from tb;
b. FROM clause
    Dùng để xác định query source.
    -- Có thể đặt alias cho table. Dùng keyword as.
        SELECT * FROM tb1 AS tt, tb2 AS bb;
    -- Sau from clause có thể xuất hiện đồng thời nhiều table.
        -- Nhiều table sẽ được xếp chồng theo chiều ngang, còn data tạo thành Cartesian product.
        SELECT * FROM tb1, tb2;
    -- Gợi ý cho optimizer cách chọn index
        USE INDEX, IGNORE INDEX, FORCE INDEX
        SELECT * FROM table1 USE INDEX (key1,key2) WHERE key1=1 AND key2=2 AND key3=3;
        SELECT * FROM table1 IGNORE INDEX (key3) WHERE key1=1 AND key2=2 AND key3=3;
c. WHERE clause
    -- Filter data source lấy được từ from.
    -- Integer 1 biểu thị true, 0 biểu thị false.
    -- Expression gồm operator và operand.
        -- Operand: variable (field), value, function return value
        -- Operator:
            =, <=>, <>, !=, <=, <, >=, >, !, &&, ||,
            in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
            is/is not kết hợp true/false/unknown để kiểm tra một value là true hay false
            <=> có chức năng giống <>, <=> có thể dùng để so sánh null
d. GROUP BY clause, group clause
    GROUP BY field/alias [sort method]
    Sau khi group sẽ tiến hành sort. Ascending: ASC, descending: DESC
    Các [aggregate function] sau cần kết hợp với GROUP BY:
    count trả về số lượng non-NULL value khác nhau  count(*), count(field)
    sum tính tổng
    max lấy giá trị lớn nhất
    min lấy giá trị nhỏ nhất
    avg tính average value
    group_concat trả về string result có non-NULL value được nối từ một group. Nối string trong group.
e. HAVING clause, condition clause
    Giống where về chức năng và cách dùng, khác thời điểm thực thi.
    where kiểm tra data lúc bắt đầu và filter data gốc.
    having filter result đã được lọc thêm lần nữa.
    Field trong having phải là field được query, field trong where phải tồn tại trong table.
    where không thể dùng field alias, having có thể. Vì lúc thực thi WHERE code, column value có thể chưa được xác định.
    where không thể dùng aggregate function. Thông thường chỉ dùng having khi cần aggregate function.
    SQL standard yêu cầu HAVING phải tham chiếu column trong GROUP BY clause hoặc column được dùng trong aggregate function.
f. ORDER BY clause, sort clause
    order by sort_field/alias sort_method [,sort_field/alias sort_method]...
    Ascending: ASC, descending: DESC
    Hỗ trợ sort nhiều field.
g. LIMIT clause, clause giới hạn số lượng result
    Chỉ giới hạn số lượng trên result đã xử lý. Coi result đã xử lý là một set, index bắt đầu từ 0 theo thứ tự record xuất hiện.
    limit start_position, row_count
    Bỏ qua parameter đầu tiên nghĩa là bắt đầu từ index 0. limit row_count
h. DISTINCT, ALL option
    distinct loại bỏ record trùng lặp
    Mặc định là all, tất cả record
```

### UNION

```sql
/* UNION */ ------------------
      Kết hợp result của nhiều SELECT query thành một result set.
      SELECT ... UNION [ALL|DISTINCT] SELECT ...
      Mặc định là DISTINCT, tức tất cả row trả về đều unique
      Khuyến nghị bọc mỗi SELECT query bằng dấu ngoặc tròn.
      Khi ORDER BY sort, cần thêm LIMIT để kết hợp.
      Số lượng field của mỗi SELECT query phải giống nhau.
      Field list (số lượng, type) của mỗi SELECT query nên nhất quán, vì field name trong result lấy theo SELECT statement đầu tiên.
```

### Subquery

```sql
/* Subquery */ ------------------
    - Subquery cần được bọc bằng dấu ngoặc.
-- FROM type
    Sau from phải là một table, bắt buộc đặt alias cho subquery result.
    - Đơn giản hóa condition trong từng query.
    - FROM type tạo result thành một temporary table, có thể dùng để giải phóng lock của table gốc.
    - Subquery trả về một table, là table-type subquery.
    select * from (select * from tb where id>0) as subfrom where id>1;
-- WHERE type
    - Subquery trả về một value, là scalar subquery.
    - Không cần đặt alias cho subquery.
    - Table trong WHERE subquery không thể trực tiếp dùng để update.
    select * from tb where money = (select max(money) from tb);
    -- Column subquery
        Nếu subquery result trả về một column.
        Dùng in hoặc not in để hoàn thành query
        exists và not exists condition
            Nếu subquery trả về data thì trả về 1 hoặc 0. Thường dùng để kiểm tra condition.
            select column1 from t1 where exists (select * from t2);
    -- Row subquery
        Query condition là một row.
        select * from t1 where (id, gender) in (select id, gender from t2);
        Row constructor: (col1, col2, ...) hoặc ROW(col1, col2, ...)
        Row constructor thường dùng để so sánh với subquery có thể trả về từ hai column trở lên.
    -- Special operator
        != all()    Tương đương not in
        = some()    Tương đương in. any là alias của some
        != some()   Không tương đương not in, khác một trong các value.
        all, some có thể kết hợp với operator khác.
```

### Join query (join)

```sql
/* Join query (join) */ ------------------
    Kết nối field của nhiều table và có thể chỉ định condition kết nối.
-- Inner join (inner join)
    - Mặc định là inner join, có thể bỏ qua inner.
    - Chỉ khi data tồn tại mới có thể thực hiện join. Nghĩa là join result không thể xuất hiện empty row.
    on biểu thị join condition. Condition expression tương tự where. Cũng có thể bỏ qua condition (biểu thị condition luôn đúng)
    Cũng có thể dùng where để biểu thị join condition.
    Còn có using nhưng yêu cầu field name giống nhau. using(field_name)
    -- Cross join
        Tức inner join không có condition.
        select * from tb1 cross join tb2;
-- Outer join (outer join)
    - Nếu data không tồn tại thì vẫn xuất hiện trong join result.
    -- Left outer join
        Nếu data không tồn tại, record của left table xuất hiện, còn right table được bổ sung null
    -- Right outer join
        Nếu data không tồn tại, record của right table xuất hiện, còn left table được bổ sung null
-- Natural join (natural join)
    Tự động xác định join condition để hoàn thành join.
    Tương đương bỏ qua using, tự động tìm field name giống nhau.
    natural join
    natural left join
    natural right join
select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;
```

### TRUNCATE

```sql
/* TRUNCATE */ ------------------
TRUNCATE [TABLE] tbl_name
Xóa sạch data
Xóa rồi tạo lại table
Khác biệt:
1, truncate xóa table rồi tạo lại, delete xóa từng row
2, truncate reset giá trị auto_increment, còn delete thì không
3, truncate không biết đã xóa bao nhiêu row, còn delete thì biết.
4, khi dùng cho table có partition, truncate giữ lại partition
```

### Backup và restore

```sql
/* Backup và restore */ ------------------
Backup, lưu structure của data và data trong table.
Dùng command mysqldump để hoàn thành.
-- Export
mysqldump [options] db_name [tables]
mysqldump [options] ---database DB1 [DB2 DB3...]
mysqldump [options] --all--database
1. Export một table
    mysqldump -uusername -ppassword database_name table_name > filename(D:/a.sql)
2. Export nhiều table
    mysqldump -uusername -ppassword database_name table1 table2 table3 > filename(D:/a.sql)
3. Export tất cả table
    mysqldump -uusername -ppassword database_name > filename(D:/a.sql)
4. Export một database
    mysqldump -uusername -ppassword --lock-all-tables --database database_name > filename(D:/a.sql)
Có thể dùng -w để truyền WHERE condition
-- Import
1. Khi đang đăng nhập MySQL:
    source backup_file
2. Khi không đăng nhập
    mysql -uusername -ppassword database_name < backup_file
```

### View

```sql
View là gì:
    View là một virtual table, nội dung do query định nghĩa. Giống table thật, view chứa một loạt column và row data có name. Tuy nhiên view không tồn tại trong database dưới dạng một tập data value được lưu. Column và row data đến từ table được query dùng để định nghĩa view, và được tạo động khi tham chiếu view.
    View có table structure file nhưng không có data file.
    Với base table được tham chiếu, view có tác dụng tương tự filter. Filter định nghĩa view có thể đến từ một hoặc nhiều table của database hiện tại hoặc database khác, hoặc từ view khác. Query qua view không có giới hạn, còn giới hạn khi sửa data qua view cũng rất ít.
    View là một SQL statement query được lưu trong database. View chủ yếu có hai lý do: lý do bảo mật, view có thể ẩn một phần data, ví dụ với table quỹ bảo hiểm xã hội có thể dùng view chỉ hiển thị name, address mà không hiển thị số bảo hiểm xã hội và salary; lý do khác là giúp query phức tạp dễ hiểu và dễ dùng.
-- Tạo view
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
    - View name phải unique, đồng thời không được trùng table name.
    - View có thể dùng column name query được bằng SELECT statement hoặc tự chỉ định column name tương ứng.
    - Có thể chỉ định algorithm thực thi view bằng ALGORITHM.
    - Nếu column_list tồn tại thì số lượng phải bằng số column SELECT statement truy vấn được
-- Xem structure
    SHOW CREATE VIEW view_name
-- Xóa view
    - Sau khi xóa view, data vẫn tồn tại.
    - Có thể xóa nhiều view cùng lúc.
    DROP VIEW [IF EXISTS] view_name ...
-- Sửa view structure
    - Thông thường không sửa view vì không phải mọi update view đều map được vào table.
    ALTER VIEW view_name [(column_list)] AS select_statement
-- Tác dụng của view
    1. Đơn giản hóa business logic
    2. Ẩn table structure thật với client
-- View algorithm (ALGORITHM)
    MERGE       Merge
        Trước tiên merge view query statement với external query rồi mới thực thi!
    TEMPTABLE   Temporary table
        Sau khi thực thi view xong sẽ tạo thành temporary table rồi thực hiện outer query!
    UNDEFINED   Undefined (mặc định), nghĩa là MySQL tự chọn algorithm tương ứng.
```

### Transaction (transaction)

```sql
Transaction là một nhóm thao tác logic; các unit tạo thành nhóm thao tác này hoặc thành công toàn bộ hoặc thất bại toàn bộ.
    - Hỗ trợ thành công hoặc rollback tập thể của nhiều SQL liên tiếp.
    - Transaction là một tính năng của database về data integrity.
    - Cần dùng InnoDB hoặc BDB storage engine để hoàn tất việc hỗ trợ auto-commit feature.
    - InnoDB được gọi là transaction-safe engine.
-- Mở transaction
    START TRANSACTION; hoặc BEGIN;
    Sau khi mở transaction, mọi SQL statement được thực thi đều được coi là SQL statement trong transaction hiện tại.
-- Commit transaction
    COMMIT;
-- Rollback transaction
    ROLLBACK;
    Nếu một phần thao tác xảy ra vấn đề thì quay về trạng thái trước khi mở transaction.
-- Transaction properties
    1. Atomicity
        Transaction là một work unit không thể chia nhỏ; hoặc mọi thao tác trong transaction đều xảy ra, hoặc không thao tác nào xảy ra.
    2. Consistency
        Data integrity trước và sau transaction phải nhất quán.
        - Data bên ngoài nhất quán tại thời điểm bắt đầu và kết thúc transaction
        - Thao tác diễn ra liên tục trong toàn bộ transaction
    3. Isolation
        Khi nhiều user truy cập database đồng thời, transaction của một user không được can thiệp bởi transaction của user khác; data giữa nhiều concurrent transaction phải được isolate.
    4. Durability
        Một khi transaction được commit, thay đổi của nó đối với data trong database là vĩnh viễn.
-- Transaction implementation
    1. Yêu cầu table type có hỗ trợ transaction
    2. Mở transaction trước khi thực thi một nhóm thao tác liên quan
    3. Sau khi cả nhóm thao tác hoàn thành, nếu thành công toàn bộ thì commit; nếu có failure thì chọn rollback để quay về backup point lúc bắt đầu transaction.
-- Transaction principle
    Dùng auto-commit (autocommit) feature của InnoDB để hoàn thành.
    Sau khi MySQL thông thường thực thi statement, thao tác commit data hiện tại có thể được client khác nhìn thấy.
    Còn transaction tạm thời tắt cơ chế “auto-commit”, cần commit để persist thao tác data.
-- Lưu ý
    1. Data Definition Language (DDL) statement không thể rollback, ví dụ statement tạo hoặc hủy database, và statement tạo, hủy hoặc thay đổi table hay stored subprogram.
    2. Transaction không thể lồng nhau
-- Savepoint
    SAVEPOINT savepoint_name -- Đặt một transaction savepoint
    ROLLBACK TO SAVEPOINT savepoint_name -- Rollback về savepoint
    RELEASE SAVEPOINT savepoint_name -- Xóa savepoint
-- Thiết lập InnoDB auto-commit feature
    SET autocommit = 0|1;   0 là tắt auto-commit, 1 là bật auto-commit.
    - Nếu tắt thì result của thao tác thông thường cũng không hiển thị với client khác; phải commit mới persist thao tác data.
    - Cũng có thể tắt auto-commit để mở transaction. Tuy nhiên khác START TRANSACTION ở chỗ:
        SET autocommit thay đổi vĩnh viễn setting của server cho đến khi setting này được sửa lại lần sau (áp dụng cho connection hiện tại)
        Còn START TRANSACTION ghi nhận state trước khi mở; sau khi transaction commit hoặc rollback thì cần mở transaction lại (áp dụng cho transaction hiện tại)

```

### Lock table

```sql
/* Lock table */
Table lock chỉ dùng để ngăn client khác đọc và ghi trái phép
MyISAM hỗ trợ table lock, InnoDB hỗ trợ row lock
-- Lock
    LOCK TABLES tbl_name [AS alias]
-- Unlock
    UNLOCK TABLES
```

### Trigger

```sql
/* Trigger */ ------------------
    Trigger là database object có name gắn với table; khi table xảy ra event cụ thể, object này sẽ được activate
    Listen: insert, update, delete record.
-- Tạo trigger
CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
    Parameter:
    trigger_time là thời điểm trigger action. Có thể là before hoặc after để chỉ trigger được kích hoạt trước hay sau statement kích hoạt nó.
    trigger_event chỉ loại statement kích hoạt trigger
        INSERT: activate trigger khi insert row mới vào table
        UPDATE: activate trigger khi thay đổi một row
        DELETE: activate trigger khi xóa một row khỏi table
    tbl_name: table được listen, phải là permanent table, không thể liên kết trigger với TEMPORARY table hoặc view.
    trigger_stmt: statement được thực thi khi trigger được activate. Để thực thi nhiều statement có thể dùng cấu trúc compound statement BEGIN...END
-- Delete
DROP TRIGGER [schema_name.]trigger_name
Có thể dùng old và new để thay thế data cũ và data mới
    Với update, trước khi update là old, sau khi update là new.
    Với delete, chỉ có old.
    Với insert, chỉ có new.
-- Lưu ý
    1. Với một table cụ thể có cùng trigger action time và event, không thể có hai trigger.
-- String concatenation function
concat(str1,str2,...])
concat_ws(separator,str1,str2,...)
-- Branch statement
if condition then
    execute statement
elseif condition then
    execute statement
else
    execute statement
end if;
-- Sửa statement delimiter của outermost statement
delimiter custom_delimiter
    SQL statement
custom_delimiter
delimiter ;     -- Đổi lại semicolon ban đầu
-- Bọc statement block
begin
    statement block
end
-- Special execution
1. Chỉ cần insert record là trigger sẽ được kích hoạt.
2. Cú pháp Insert into on duplicate key update sẽ trigger:
    Nếu không có duplicate record thì trigger before insert, after insert;
    Nếu có duplicate record và update thì trigger before insert, before update, after update;
    Nếu có duplicate record nhưng không xảy ra update thì trigger before insert, before update
3. Cú pháp Replace: nếu có record thì thực thi before insert, before delete, after delete, after insert
```

### SQL programming

```sql
/* SQL programming */ ------------------
--// Local variable ----------
-- Variable declaration
    declare var_name[,...] type [default value]
    Statement này dùng để khai báo local variable. Để cung cấp default value cho variable, hãy thêm default clause. Value có thể được chỉ định bằng expression, không cần là constant. Nếu không có default clause thì initial value là null.
-- Assignment
    Dùng statement set và select into để gán value cho variable.
    - Lưu ý: Có thể dùng global variable (user-defined variable) trong function
--// Global variable ----------
-- Definition, assignment
    Statement set có thể định nghĩa và gán value cho variable.
    set @var = value;
    Cũng có thể dùng statement select into để initialize và gán value cho variable. Khi đó select statement chỉ được trả về một row, nhưng có thể có nhiều field, nghĩa là có thể gán value cho nhiều variable cùng lúc; số lượng variable phải khớp số column của query.
    Cũng có thể coi assignment statement là một expression và thực thi bằng select. Khi đó để tránh = bị coi là relational operator, dùng := thay thế. (set statement có thể dùng = và :=).
    select @var:=20;
    select @v1:=id, @v2=name from t1 limit 1;
    select * from tbl_name where @var:=30;
    select into có thể gán data query được từ table cho variable.
        -| select max(height) into @max_height from tb;
-- Custom variable name
    Để tránh user-defined variable trong SELECT statement xung đột với system identifier (thường là field name), user-defined variable dùng @ làm symbol bắt đầu trước variable name.
    @var=10;
        - Sau khi được định nghĩa, variable có hiệu lực trong toàn bộ session (từ lúc login đến lúc logout)
--// Control structure ----------
-- if statement
if search_condition then
    statement_list
[elseif search_condition then
    statement_list]
...
[else
    statement_list]
end if;
-- case statement
CASE value WHEN [compare-value] THEN result
[WHEN [compare-value] THEN result ...]
[ELSE result]
END
-- while loop
[begin_label:] while search_condition do
    statement_list
end while [end_label];
- Nếu cần kết thúc while loop sớm bên trong loop thì cần dùng label; label phải xuất hiện theo cặp.
    -- Exit loop
        Thoát toàn bộ loop: leave
        Thoát loop hiện tại: iterate
        Dùng exit label để quyết định thoát loop nào
--// Built-in function ----------
-- Numeric function
abs(x)          -- Absolute value abs(-10.9) = 10
format(x, d)    -- Format numeric value with thousands separator format(1234567.456, 2) = 1,234,567.46
ceil(x)         -- Round up ceil(10.1) = 11
floor(x)        -- Round down floor (10.1) = 10
round(x)        -- Round to integer
mod(m, n)       -- m%n m mod n remainder 10%3=1
pi()            -- Get pi
pow(m, n)       -- m^n
sqrt(x)         -- Arithmetic square root
rand()          -- Random number
truncate(x, d)  -- Truncate d decimal places
-- Date and time function
now(), current_timestamp();     -- Current date and time
current_date();                 -- Current date
current_time();                 -- Current time
date('yyyy-mm-dd hh:ii:ss');    -- Get date part
time('yyyy-mm-dd hh:ii:ss');    -- Get time part
date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j'); -- Format time
unix_timestamp();               -- Get Unix timestamp
from_unixtime();                -- Get time from timestamp
-- String function
length(string)          -- String length in byte
char_length(string)     -- Number of characters in string
substring(str, position [,length])      -- Get length characters from position in str
replace(str ,search_str ,replace_str)   -- Replace search_str with replace_str in str
instr(string ,substring)    -- Return the first position of substring in string
concat(string [,...])   -- Concatenate string
charset(str)            -- Return string character set
lcase(string)           -- Convert to lowercase
left(string, length)    -- Get length characters from the left of string2
load_file(file_name)    -- Read content from file
locate(substring, string [,start_position]) -- Same as instr, but can specify start position
lpad(string, length, pad)   -- Repeat pad at the beginning of string until string length is length
ltrim(string)           -- Remove leading spaces
repeat(string, count)   -- Repeat count times
rpad(string, length, pad)   -- Add pad after str until length is reached
rtrim(string)           -- Remove trailing spaces
strcmp(string1 ,string2)    -- Compare two strings character by character
-- Flow function
case when [condition] then result [when [condition] then result ...] [else result] end   Multiple branches
if(expr1,expr2,expr3)  Two branches.
-- Aggregate function
count()
sum();
max();
min();
avg();
group_concat()
-- Other common function
md5();
default();
--// Stored function, custom function ----------
-- Create
    CREATE FUNCTION function_name (parameter_list) RETURNS return_type
        function_body
    - Function name phải là identifier hợp lệ và không nên xung đột với keyword có sẵn.
    - Function phải thuộc về một database. Có thể thực thi function hiện thuộc database bằng dạng db_name.function_name; nếu không thì dùng database hiện tại.
    - Parameter gồm "parameter name" và "parameter type". Nhiều parameter ngăn cách bằng dấu phẩy.
    - Function body gồm nhiều MySQL statement, control flow, variable declaration và các statement khác có thể dùng.
    - Nhiều statement cần được bọc bằng statement block begin...end.
    - Bắt buộc có statement return value.
-- Delete
    DROP FUNCTION [IF EXISTS] function_name;
-- View
    SHOW FUNCTION STATUS LIKE 'partten'
    SHOW CREATE FUNCTION function_name;
-- Modify
    ALTER FUNCTION function_name function_options
--// Stored procedure, custom function ----------
-- Definition
    Stored procedure là một đoạn code (procedure), gồm SQL được lưu trong database.
    Stored procedure thường dùng để hoàn thành một đoạn business logic, ví dụ đăng ký, thu tiền lớp, nhập order vào database.
    Function thường tập trung vào một chức năng; được coi là phục vụ program khác và phải được gọi trong statement khác, còn stored procedure không thể được gọi bởi bên khác mà tự thực thi thông qua call.
-- Create
CREATE PROCEDURE sp_name (parameter_list)
    procedure_body
Parameter list: khác parameter list của function ở chỗ cần chỉ định parameter type
IN, biểu thị input
OUT, biểu thị output
INOUT, biểu thị input và output
Lưu ý, không có return value.
```

### Stored procedure

```sql
/* Stored procedure */ ------------------
Stored procedure là một tập hợp executable code. So với function, nó thiên về business logic hơn.
Call: CALL procedure_name
-- Lưu ý
- Không có return value.
- Chỉ có thể gọi độc lập, không thể chèn vào statement khác
-- Parameter
IN|OUT|INOUT parameter_name data_type
IN      Input: trong quá trình call, input data vào parameter bên trong procedure body
OUT     Output: trong quá trình call, trả result đã được procedure body xử lý về client
INOUT   Input output: vừa có thể input vừa có thể output
-- Syntax
CREATE PROCEDURE procedure_name (parameter_list)
BEGIN
    procedure_body
END
```

### User và permission management

```sql
/* User và permission management */ ------------------
-- Reset root password
1. Dừng MySQL service
2.  [Linux] /usr/local/mysql/bin/safe_mysqld --skip-grant-tables &
    [Windows] mysqld --skip-grant-tables
3. use mysql;
4. UPDATE `user` SET PASSWORD=PASSWORD("password") WHERE `user` = "root";
5. FLUSH PRIVILEGES;
User information table: mysql.user
-- Refresh permission
FLUSH PRIVILEGES;
-- Add user
CREATE USER username IDENTIFIED BY [PASSWORD] password(string)
    - Phải có global CREATE USER permission của mysql database hoặc có INSERT permission.
    - Chỉ tạo được user, không thể cấp permission.
    - Username, lưu ý quote: ví dụ 'user_name'@'192.168.1.1'
    - Password cũng cần quote, password chỉ gồm số cũng phải thêm quote
    - Để chỉ định password dưới dạng plain text, bỏ qua keyword PASSWORD. Để chỉ định password là giá trị hash do function PASSWORD() trả về, phải thêm keyword PASSWORD
-- Rename user
RENAME USER old_user TO new_user
-- Set password
SET PASSWORD = PASSWORD('password')  -- Set password cho user hiện tại
SET PASSWORD FOR username = PASSWORD('password') -- Set password cho user được chỉ định
-- Delete user
DROP USER username
-- Grant permission/add user
GRANT permission_list ON table_name TO username [IDENTIFIED BY [PASSWORD] 'password']
    - all privileges biểu thị tất cả permission
    - *.* biểu thị tất cả table của tất cả database
    - database_name.table_name biểu thị table cụ thể trong database
    GRANT ALL PRIVILEGES ON `pms`.* TO 'pms'@'%' IDENTIFIED BY 'pms0817';
-- View permission
SHOW GRANTS FOR username
    -- View permission của user hiện tại
    SHOW GRANTS; hoặc SHOW GRANTS FOR CURRENT_USER; hoặc SHOW GRANTS FOR CURRENT_USER();
-- Revoke permission
REVOKE permission_list ON table_name FROM username
REVOKE ALL PRIVILEGES, GRANT OPTION FROM username   -- Revoke tất cả permission
-- Permission level
-- Để dùng GRANT hoặc REVOKE, bạn phải có GRANT OPTION permission và phải có permission mà bạn đang grant hoặc revoke.
Global level: global permission áp dụng cho mọi database trong một server đã cho, mysql.user
    GRANT ALL ON *.* và REVOKE ALL ON *.* chỉ grant và revoke global permission.
Database level: database permission áp dụng cho mọi object trong một database đã cho, mysql.db, mysql.host
    GRANT ALL ON db_name.* và REVOKE ALL ON db_name.* chỉ grant và revoke database permission.
Table level: table permission áp dụng cho mọi column trong một table đã cho, mysql.tables_priv
    GRANT ALL ON db_name.tbl_name và REVOKE ALL ON db_name.tbl_name chỉ grant và revoke table permission.
Column level: column permission áp dụng cho một column trong một table đã cho, mysql.columns_priv
    Khi dùng REVOKE, phải chỉ định các column giống với column đã được cấp permission.
-- Permission list
ALL [PRIVILEGES]    -- Set tất cả simple permission, trừ GRANT OPTION
ALTER   -- Cho phép dùng ALTER TABLE
ALTER ROUTINE   -- Thay đổi hoặc hủy stored subprogram
CREATE  -- Cho phép dùng CREATE TABLE
CREATE ROUTINE  -- Tạo stored subprogram
CREATE TEMPORARY TABLES     -- Cho phép dùng CREATE TEMPORARY TABLE
CREATE USER     -- Cho phép dùng CREATE USER, DROP USER, RENAME USER và REVOKE ALL PRIVILEGES.
CREATE VIEW     -- Cho phép dùng CREATE VIEW
DELETE  -- Cho phép dùng DELETE
DROP    -- Cho phép dùng DROP TABLE
EXECUTE     -- Cho phép user chạy stored subprogram
FILE    -- Cho phép dùng SELECT...INTO OUTFILE và LOAD DATA INFILE
INDEX   -- Cho phép dùng CREATE INDEX và DROP INDEX
INSERT  -- Cho phép dùng INSERT
LOCK TABLES     -- Cho phép dùng LOCK TABLES trên table mà user có SELECT permission
PROCESS     -- Cho phép dùng SHOW FULL PROCESSLIST
REFERENCES  -- Chưa được triển khai
RELOAD  -- Cho phép dùng FLUSH
REPLICATION CLIENT  -- Cho phép user hỏi địa chỉ của replica server hoặc primary server
REPLICATION SLAVE   -- Dùng cho replica server (đọc binary log event từ primary server)
SELECT  -- Cho phép dùng SELECT
SHOW DATABASES  -- Hiển thị tất cả database
SHOW VIEW   -- Cho phép dùng SHOW CREATE VIEW
SHUTDOWN    -- Cho phép dùng mysqladmin shutdown
SUPER   -- Cho phép dùng CHANGE MASTER, KILL, PURGE MASTER LOGS và SET GLOBAL statement, mysqladmin debug command; cho phép kết nối một lần dù đã đạt max_connections.
UPDATE  -- Cho phép dùng UPDATE
USAGE   -- Alias của “no permission”
GRANT OPTION    -- Cho phép cấp permission
```

### Bảo trì table

```sql
/* Bảo trì table */
-- Phân tích và lưu distribution của table keyword
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE table_name ...
-- Kiểm tra một hoặc nhiều table có lỗi hay không
CHECK TABLE tbl_name [, tbl_name] ... [option] ...
option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}
-- Sắp xếp fragment của data file
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

### Miscellaneous

```sql
/* Miscellaneous */ ------------------
1. Có thể dùng backtick (`) để bọc identifier (database name, table name, field name, index, alias), tránh trùng keyword! Cũng có thể dùng tiếng Việt làm identifier!
2. Mỗi database directory có một option file db.opt lưu option của database hiện tại.
3. Comment:
    Single-line comment # comment content
    Multi-line comment /* comment content */
    Single-line comment -- comment content (standard SQL comment style, yêu cầu thêm một space character (space, TAB, newline, v.v.) sau hai dấu gạch ngang)
4. Pattern wildcard:
    _   Bất kỳ một character nào
    %   Bất kỳ số lượng character nào, kể cả zero character
    Single quote cần escape \'
5. Statement delimiter trong CMD command line có thể là ";", "\G", "\g", chỉ ảnh hưởng result hiển thị. Ở nơi khác vẫn kết thúc bằng semicolon. delimiter có thể sửa statement delimiter của dialogue hiện tại.
6. SQL không phân biệt uppercase/lowercase
7. Xóa statement hiện có: \c
```

<!-- @include: @article-footer.snippet.md -->
