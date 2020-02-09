# sqlalchemy 涉及到的知识点

## sqlalchemy.ext.declarative.declarative_base



## sqlalchemy.orm.relationship

### 引用到的包

```python
from sqlalchemy import Table, Column, Integer, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

### 一对多

父节点和子节点

```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    children = relationship("Child")

class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
    parent_id = Column(Integer, ForeignKey('parent.id'))
```

metadata型,declarative型

column属性:

​	数据类型:

关系

​	

json to base

属性: 

```json
{"tablename":"base","columns":"[{"name":"xxx":}]","relationships":"xxx"}
```

gua

