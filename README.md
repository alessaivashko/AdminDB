Создаем основную БД:

create database DB;
go
use DB;
go

drop view if exists dbo.getNewID;
drop function if exists dbo.myrand;
drop function if exists dbo.randpass;
go

Создаем несколько пользователей с рандомными паролями:

create view getNewID as select newid() as new_id
go

create function myrand ()
returns uniqueidentifier
as begin
return (select new_id from getNewID)
end
go

Функция для создания рандомного пароля:

create function randpass @lenn smallint)
returns varchar(200) as
begin

declare @res varchar(200) = (
select
string_agg(letter,'')
from (
select top @lenn)
char(abs(checksum(dbo.myrand())) % 43 + ascii('0')) letter
from
sys.all_objects) t)

return @res
end
go

Создаем таблицу для хранения логинов и паролей:

create table Users (login varchar(200), password varbinary(max));
go

Теперь нужно зашифровать пароли:

create master key encryption by password = 'password'
go

create asymmetric key ak
with algorithm = rsa_2048
encryption by password = 'password'
go

create symmetric key sk
with algorithm = aes_256
encryption by asymmetric key ak
go

А теперь нужно расшифровать пароли:

open symmetric key sk
decryption by asymmetric key ak
with password = 'password'

declare @l smallint = 1
while @l <= 3
begin
declare @login varchar(10) = 'l'+cast (@l as varchar(10))
declare @password varchar(200) = dbo.randpass(5)
EXEC sp_addlogin @login, @password
insert Users values @loginn, ENCRYPTBYKEY(KEY_GUID('sk'),@password) )
select @login, @password
set @l += 1
end
go

close symmetric key sk

Создаем 3 дополнительные БД:

create database DB1;
create database DB2;
create database DB3;

Назначаем роли наших пользователей:

use DB1;
create user user1 for login l1;
EXEC sp_addrolemember 'db_owner', 'user1'
use DB2;
create user user2 for login l2;
EXEC sp_addrolemember 'db_owner', 'user2'
use DB3;
create user user3 for login l3;
EXEC sp_addrolemember 'db_owner', 'user3'

use DB;

open symmetric key sk
decryption by asymmetric key ak
with password = 'password'

select login, convert(varchar(32), DECRYPTBYKEY([password])) from Users

close symmetric key sk

Создание бэкап файла БД:

BACKUP DATABASE [DB] TO DISK = N'C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Backup\demo_succesful_copied.bak'
WITH NOFORMAT, NOINIT, NAME = N'DB - bak', SKIP, NOREWIND, NOUNLOAD, STATS = 10
GO

Восстановление БД из бэкапа:

RESTORE DATABASE DB
FROM DISK = N'C:\Program Files\Microsoft SQL Server\MSSQL15.MSSQLSERVER\MSSQL\Backup\demo_succesful_copied.bak'

Создаем процедуру для проверки корректности ввода почты:

create procedure regexing as (select email, case when email like '%%@%%.%_%' and email not like '%[["<>'']%' then 1 else 0 end valid
from Users twhere email is not null
)go
exec dbo.regexing
