## django模型之间的关联  
含有关系字段的表称为子表，被关联的表称为母表
### 一对一关系  
一对一关系可以将关系字段定义在任意一方  
```python
class Student(models.Model):
    stuname = models.CharField(max_length=20)
    stusex = models.IntegerField()
    stubirth = models.DateField()
    stutel = models.CharField(max_length=255)

    class Meta:
        db_table = 'student'


class StuInfo(models.Model):
    stu_addr = models.CharField(max_length=20)
    stu_age = models.IntegerField()
    stu = models.OneToOneField(Student)

    class Meta:
        db_table = 'stu_info'
```  
这里stu_info称为子表，student称为母表  
正向查询，即从子表查询母表，有两种形式  
一种是  
```python  
# 正向查询，子表查询母表，第一种方式
    stuinfo=StuInfo.objects.get(id=1)
    print(stuinfo.stu.stuname)
# 子表对象.子表关联的字段(如果存在related_name，则使用related_name).母表字段名
```  
另一种是
```python
student=Student.objects.get(stuinfo__id=1)
print(student.stuname)
#母表.objects.get(子表表名小写__子表字段='xxx').母表字段
```  
反向查询，即从母表查询子表，有两种形式  
一种是  
```python
student = Student.objects.get(id=1)
print(student.stuinfo.stu_addr)
# 母表对象.子表表名小写.子表字段
```  
另一种是  
```python
stuinfo = StuInfo.objects.get(stu_id__id=1)
print(stuinfo.stu_addr)
# 子表类名.objects.get(子表关联字段__母表字段='xxx').子表字段
```

