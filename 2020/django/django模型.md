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
stuinfo = StuInfo.objects.first()
    # 获得对应的母表对象
    student = stuinfo.stu
    # 查询该对象的stuname属性
    print(stuinfo.stuname)
# 
```  
另一种是
```python
student=Student.objects.get(stuinfo__id=1)
print(student.stuname)
```  
反向查询，即从母表查询子表，有两种形式  
一种是  
```python
 # 获得一个母表对象
 student=Student.objects.get(id=1)
 # 获得子表对象
 # 这里获取子表对象默认使用子表名称的小写，如果指定里related_name,则使用related_name
 studentInfo=student.stuinfo
 # 获得子表对象的属性
 print(studentInfo.stu_addr)
```  
另一种是  
```python
stuinfo = StuInfo.objects.get(stu_id__id=1)
print(stuinfo.stu_addr)
```

