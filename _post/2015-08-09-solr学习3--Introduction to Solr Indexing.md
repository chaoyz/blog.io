# [Introduction to Solr Indexing](https://cwiki.apache.org/confluence/display/solr/Introduction+to+Solr+Indexing)

This section describes the process of indexing: adding content to a Solr index and, if necessary, modifying that content or deleting it. By adding content to an index, we make it searchable by Solr.

这一章节描述了建立索引的过程：向索引中添加内容，修改或者删除内容的索引。通过向索引中添加内容，我们才能使用solr进行搜索。（= =说的有点拗口，意思就是不添加内容你还搜索个毛）

A Solr index can accept data from many different sources, including XML files, comma-separated value (CSV) files, data extracted from tables in a database, and files in common file formats such as Microsoft Word or PDF.

solr的索引可以接收不同的数据格式，包括xml文件，csv文件，数据库中的表，或者是常见的文件格式如word和pdf

Here are the three most common ways of loading data into a Solr index:

- Using the [Solr Cell](https://cwiki.apache.org/confluence/display/solr/Uploading+Data+with+Solr+Cell+using+Apache+Tika) framework built on Apache Tika for ingesting binary files or structured files such as Office, Word, PDF, and other proprietary formats.

- Uploading XML files by sending HTTP requests to the Solr server from any environment where such requests can be generated.
- Writing a custom Java application to ingest data through Solr's Java Client API (which is described in more detail in[Client APIs](https://cwiki.apache.org/confluence/display/solr/Client+APIs). Using the Java API may be the best choice if you're working with an application, such as a Content Management System (CMS), that offers a Java API.

通常有三种方式向solr中添加索引

1、使用solr子框架（= =不知道对不对）Tika提取二进制文件或者结构化的文档如office、word、pdf和一些其他格式的格式文档（MP4、MP3都可以~~~，Tika这玩意很强大能提取很多格式文档的元数据！！！）

2、通过solr的java api（solrj）向solr服务器发送Http请求上传xml格式的数据。

3、编写一个java程序，使用solr的java api（solrj）。详情请见Client APIs章节。当你想使用solr功能完善你自己java程序的时候，使用java api是一个不错的选择，（= =这里不会翻译了）

Regardless of the method used to ingest data, there is a common basic data structure for data being fed into a Solr index: a document containing multiple fields, each with a name and containing content, which may be empty. One of the fields is usually designated as a unique ID field (analogous to a primary key in a database), although the use of a unique ID field is not strictly required by Solr.

无论你用什么方式向solr提供数据，向solr提供的数据必须是一种基础的数据格式：一个文档包含好几个字段（fileds），每一个字段都有一个name和字段包含的内容（content），（这里没确定是filed还是content）可以是空。每一个filed通常上都会被设定一个唯一的ID（可以类比一下数据库中的主键），但是solr没有强制要求必须使用唯一ID~~~

If the field name is defined in the `schema.xml` file that is associated with the index, then the analysis steps associated with that field will be applied to its content when the content is tokenized. Fields that are not explicitly defined in the schema will either be ignored or mapped to a dynamic field definition (see [Documents, Fields, and Schema Design](https://cwiki.apache.org/confluence/display/solr/Documents%2C+Fields%2C+and+Schema+Design)), if one matching the field name exists.

如果字段（field）的名字在schema.xml文件中已经指定，那么该字段（field）将会关联到索引上（= =！又不知道怎么翻译了,凑和着看吧。），在solr分析过程中，当字段（field）中的内容（content）被分词之后，字段将会与其内容关联起来。如果fields没有在schema.xml文件中明确的定义，在solr的运行中它可能会被忽略不处理，若这个字段满足schema.xml文件中定义的某一个动态字段（dynamic field）则按照定义的动态字段处理。

The Solr Example Directory

When starting Solr with the "-e" option, the example/ directory will be used as base directory for the example Solr instances that are created.  This directory also includes an example/exampledocs/ subdirectory containing sample documents in a variety of formats that you can use to experiment with indexing into the various examples.

当使用-e参数启动solr时，solr将会创建一个example实例以example/ 为根目录。在这个example/ 文件夹内包含example/exampledocs/ 子目录，在这个子目录中很多不同格式的文件，我们可以使用这些文件测试创建索引~~~

 