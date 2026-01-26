Nếu import từ file excel vào FBO mà gặp lỗi có xác định nhưng chưa đoán ra là lỗi cụ thể là gì thì ta có thể tiến hành tạo query để debug theo các bước như sau:

1.  Tại file xml trong App_Data\\Controllers\\Templates\\Upload (\*Tran.xml, \*Detail.xml, \*Master.xml…) ta thêm vào đầu thẻ &lt;processing&gt; đoạn code như sau:

***if object_id('zzzimportexcel') is not null drop table zzzimportexcel;***
***select * into zzzimportexcel from @@table;***

1.  Mở SQL Server Profiler, rồi vào FBO tiến hành import lại (tất nhiên vẫn gặp lỗi có xác định)
2.  Tìm đến đoạn create table #table và copy ra SQL Query

3.  Tìm đến đoạn code trong thẻ &lt;processing&gt; (thường bắt đầu bằng declare @ma_dvcs nvarchar(4),@status nvarchar(2)…) và copy ra SQL Query

4.  Copy đoạn code sau vào giữa 2 đoạn code trên

***alter table zzzimportexcel drop column stt;***
***insert into #table select \* from zzzimportexcel***

1.  Tiến hành Execute Query đã tạo, có thể đặt các điểm bug (select, print…) để xác định vị trí gặp lỗi

- _Nếu Execute lại mà gặp lỗi như trên thì chủ động tạo câu lệnh drop table #_
- _Nếu gặp lỗi lạ báo đỏ tương tự như trên thì có thể copy toàn bộ code hiện tại sang 1 New Query và chạy thử lại_

1.  Sau khi debug thành công vui lòng xóa hoặc comment đoạn code đã thêm ban đầu

***/\*if object_id('zzzimportexcel') is not null drop table zzzimportexcel; select \* into zzzimportexcel from @@table;\*/***
