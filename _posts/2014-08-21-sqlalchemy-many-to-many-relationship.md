---
layout: post
title: SQLAlchemy多对多关系使用方法
---

SQLAlchemy是一个Python的数据库ORM框架.

多对多数据模型是比较常用的数据关系.

下面采用文章(```Page```)和标签(```Tag```)的关系来作为介绍范例.

1. 一篇文章可以通过多个标签来描述概要信息.
1. 一个标签可以对应多篇文章.

首先我们先创建两个Sqlalchemy的模型类:

    class Page(Base):
        __tablename__ = 'page'
        id = Column(Integer, primary_key=True)
        name = Column(String)
        
    class Tag(Base):
        __tablename__ = 'tag'
        id = Column(Integer, primary_key=True)
        name = Column(String)
        
这里创建了两个模型, 分别是```Page```和```Tag```.

添加一个需求: 我们能够在```Page```对象中直接获取关联的```Tag```列表

那么我们就需要在```Page```模型中添加:

    class Page(Base):
    ...
        tags = relationship('Tag', secondary=association_table)
    ...

这里就通过指定```relationship```来指定```tags```属性是指向```Tag```对象的, 关联规则是通过```association_table```的关联关系.

然后就是需要创建关联规则表```association_table```.

    association_table = Table(
        'association', Base.metadata,
        Column('page_id', Integer, ForeignKey('page.id')),
        Column('tag_id', Integer, ForeignKey('tag.id'))
    )
    
这里就创建一个```Page-Tag```关联表, 通过将```page_id```作为```Page```对象的外键. 同理```tag_id```作为```Tag```对象的外键.

至此, 我们已经完成多对多模型的创建.

接下来就是多对多模型的CRUD了.

    page = Page(name='Python API Page')
    api_tag = Tag(name='api')
    python_tag = Tag(name='python')
    
    # append tag to page
    page.tags.append(api_tag)
    page.tags.append(python_tag)

    # read tag from page
    print [tag.name for tag in page.tags]

    # remove tag from page
    page.tags.remove(python_tag)

以上就是Sqlalchemy关于多对多模型操作方式~

下面贴上完整的代码:

    import sqlalchemy as sqla
    import sqlalchemy.orm as sqlorm
    from sqlalchemy.ext.declarative import declarative_base as sqla_declarative_base

    Base = sqla_declarative_base()
    engine = sqla.create_engine('sqlite:///test.db', echo=True)

    association_table = sqla.Table(
        'association', Base.metadata,
        sqla.Column('page_id', sqla.Integer, sqla.ForeignKey('page.id')),
        sqla.Column('tag_id', sqla.Integer, sqla.ForeignKey('tag.id'))
    )

    class Page(Base):
        __tablename__ = 'page'
        id = sqla.Column(sqla.Integer, primary_key=True)
        name = sqla.Column(sqla.String)
        tags = sqlorm.relationship('Tag', secondary=association_table)

    class Tag(Base):
        __tablename__ = 'tag'
        id = sqla.Column(sqla.Integer, primary_key=True)
        name = sqla.Column(sqla.String)
        pages = sqlorm.relationship('Page', secondary=association_table)

    Base.metadata.bind = engine
    Base.metadata.create_all()

    Session = sqlorm.scoped_session(sqlorm.sessionmaker(bind=engine))

    def save_page():
        sess = Session()
        try:
            page = Page(name='Python API Page')
            sess.add(page)

            sess.flush()
            sess.commit()
        finally:
            sess.close()

    def add_tag():
       sess = Session()
       try:
           python_tag = Tag(name='python')
           api_tag = Tag(name='api')
           sess.add(python_tag)
           sess.add(api_tag)
           page = sess.query(Page).first()
           page.tags.append(python_tag)
           page.tags.append(api_tag)

           sess.flush()
           sess.commit()
       finally:
           sess.close()

    def remove_tag():
        sess = Session()
        try:
            page = sess.query(Page).first()
            api_tag = sess.query(Tag).filter(Tag.name=='api').first()
            page.tags.remove(api_tag)
            sess.flush()
            sess.commit()
        finally:
            sess.close()

    if __name__ == '__main__':
        save_page()
        add_tag()
        remove_tag()
