## Jena
Jena
## RDF
RDF是一个三元组模型，即每一份知识都被分解成为如下形式(subject(主)，predicate(谓)，object(宾))

**VACRD**
W3C为了满足一些新需要，定义了另一套规范(http://www.w3.org/TR/vcard-rdf)，
它用RDF(Resource Description Framework)来描述vCard，表示的内容一样，只是用XML来的描述的。如：

    <vCard:Family> Crystal </vCard:Family>
    <vCard:Given>  Corky </vCard:Given>
     <vCard:Other>  Jacky </vCard:Other>
     <vCard:Prefix> Dr </vCard:Prefix>
**Jena中的VACRD**
uri: [http://www.w3.org/2001/vcard-rdf/3.0](http://www.w3.org/2001/vcard-rdf/3.0)

```
	ORGPROPERTIES： orgproperties: 组织
    ADRTYPES:       adrtypes: 
    NPROPERTIES     nproperties:
    EMAILTYPES      emailtypes
    TELTYPES        teltypes
    ADRPROPERTIES   adrpoperties
    TZTYPES         tzitypes
    Street          street
    AGENT           agent
    Source          source
    LOGO            logo
    BDAY            bady
    REV             rev
    SORT_STRING     sort_string
    Orgname         orgname
    CATEGORIES      categories
    N               n
    Pcode           pcode
    Prefix          prefix
    PHOTO           photo
    FN              fn
    ORG             org
    Suffix          suffix
    CLASS           class
    ADR             adr
    Region          region
    GEO             geo
    Extadd          extadd
    EMAIL           email
    UID             uid
    Family          family
    TZ              tz
    Name            name
    Orgunit         orgunit
    Country         country
    SOUND           sound
    Title           title
    NOTE            note
    MAILER          maller
    Other           other
    Locality        locality
    Pobox           pobox
    KEY             key
    PRODID          prodid
    Given           given
    LABEL           label
    TEL             tel
    NICKNAME        nickname
    ROLE            role
```


**简单介绍**
```Java
# 创建一个简单的模型，表明主语 personURI 谓语FullName 宾语 John Smith
# 即 主语+谓语=宾语，谓语表明两者关系，主语的xxx是谓语这样的意义
// some definitions
static String personURI    = "http://somewhere/JohnSmith";
static String fullName     = "John Smith";

// create an empty Model
Model model = ModelFactory.createDefaultModel();

// create the resource
Resource johnSmith = model.createResource(personURI);

// add the property
 johnSmith.addProperty(VCARD.FN, fullName);

```
![Markdown](http://i2.bvimg.com/654958/8c34d774b370ed3c.png)
这样的一个简单的模型就创建完毕了。

主要API
```
1 Model model = ModelFactory...	创建空模型
2 所有的节点均被称为Resource，而所有的谓语均被称为 property
3 谓语有具体的意义FN,ORG等
4 给主语添加谓语，需要给定谓语的意义和谓语后面的宾语
```
property属性中只可以增加Resource吗？也可以跟一个model

```Java
 String personURI    = "http://somewhere/JohnSmith";
        String givenName    = "John";
        String familyName   = "Smith";
        String fullName     = givenName + " " + familyName;

        // create an empty model
        Model model = ModelFactory.createDefaultModel();

        // create the resource
        //   and add the properties cascading style
        Resource johnSmith = model.createResource(personURI).addProperty(VCARD.FN,fullName)
                .addProperty(VCARD.N,model.createResource().addProperty(VCARD.Given,givenName).addProperty(VCARD.Family,familyName));

```
将model作为宾语，谓语是vacrd:N 是谓语，名字，名字分为名和姓，这样就可以更为全面的介绍了

![Markdown](http://i2.bvimg.com/654958/8f8eee20e61ced98.png)

**读取model中的数据**
**Statements**
每一个由主谓宾构成的三元组均被称为statements
A statement is sometimes called a triple, because of its three parts.

    http://somewhere/JohnSmith http://www.w3.org/2001/vcard-rdf/3.0#N 21fa9cc9-7980-4aa1-a985-59b57307dbbf .
    三列数据，分别代表 subject predicate object 

```Java
 StmtIterator iter=model.listStatements();
        while(iter.hasNext()){
            Statement stmt = iter.nextStatement();
            Resource subject = stmt.getSubject();
            Property predicate = stmt.getPredicate();
            RDFNode  object =stmt.getObject();

            System.out.print(subject.toString());
            System.out.print(" " + predicate.toString() + " ");
            if (object instanceof Resource) {
                System.out.print(object.toString());
            } else {
                // object is a literal
                System.out.print(" \"" + object.toString() + "\"");
            }
            System.out.println(" .");
        }

>>>
http://somewhere/JohnSmith http://www.w3.org/2001/vcard-rdf/3.0#N 97221207-932d-47cb-b2c5-7dedee4211fe .
http://somewhere/JohnSmith http://www.w3.org/2001/vcard-rdf/3.0#FN  "John Smith" .
97221207-932d-47cb-b2c5-7dedee4211fe http://www.w3.org/2001/vcard-rdf/3.0#Family  "Smith" .
97221207-932d-47cb-b2c5-7dedee4211fe http://www.w3.org/2001/vcard-rdf/3.0#Given  "John" .

```
输出整个模型 会将模型的结构输出，如果能输出一张类似图片的就更好了
```Java
model.write(System.out)
>>>
<rdf:RDF
    xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
    xmlns:vcard="http://www.w3.org/2001/vcard-rdf/3.0#">
  <rdf:Description rdf:about="http://somewhere/JohnSmith">
    <vcard:N rdf:parseType="Resource">
      <vcard:Family>Smith</vcard:Family>
      <vcard:Given>John</vcard:Given>
    </vcard:N>
    <vcard:FN>John Smith</vcard:FN>
  </rdf:Description>
</rdf:RDF>
```
读取模型
```rdf
<rdf:RDF
  xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'
  xmlns:vCard='http://www.w3.org/2001/vcard-rdf/3.0#'
   >

  <rdf:Description rdf:about="http://somewhere/JohnSmith/">
    <vCard:FN>John Smith</vCard:FN>
    <vCard:N rdf:parseType="Resource">
	    <vCard:Family>Smith</vCard:Family>
	    <vCard:Given>John</vCard:Given>
    </vCard:N>
  </rdf:Description>

  <rdf:Description rdf:about="http://somewhere/RebeccaSmith/">
    <vCard:FN>Becky Smith</vCard:FN>
    <vCard:N rdf:parseType="Resource">
	    <vCard:Family>Smith</vCard:Family>
	    <vCard:Given>Rebecca</vCard:Given>
    </vCard:N>
  </rdf:Description>

  <rdf:Description rdf:about="http://somewhere/SarahJones/">
    <vCard:FN>Sarah Jones</vCard:FN>
    <vCard:N rdf:parseType="Resource">
	<vCard:Family>Jones</vCard:Family>
	<vCard:Given>Sarah</vCard:Given>
    </vCard:N>
  </rdf:Description>

  <rdf:Description rdf:about="http://somewhere/MattJones/">
    <vCard:FN>Matt Jones</vCard:FN>
    <vCard:N
	vCard:Family="Jones"
	vCard:Given="Matthew"/>
  </rdf:Description>

</rdf:RDF>

```
读取解析模型
```Java
Model model = ModelFactory.createDefaultModel();

        InputStream in = FileManager.get().open( inputFileName );
        if (in == null) {
            throw new IllegalArgumentException( "File: " + inputFileName + " not found");
        }

        // read the RDF/XML file
        model.read(in, "");
        model.write(System.out,"N-TRIPLES");
>>>
_:B4bf497b4X2D76a6X2D4a5cX2D8bdaX2D7b4a0585a6b4 <http://www.w3.org/2001/vcard-rdf/3.0#Given> "Sarah" .
_:B4bf497b4X2D76a6X2D4a5cX2D8bdaX2D7b4a0585a6b4 <http://www.w3.org/2001/vcard-rdf/3.0#Family> "Jones" .

        
```

如果读取模型了，并且选择性的列出里面的statements
```Java
# 使用SimpleSelector
 // select all the resources with a VCARD.FN property
        // whose value ends with "Smith"
        StmtIterator iter = model.listStatements(
            new SimpleSelector(null,VCARD.FN,(RDFNode) null){
                @Override
                public boolean selects(Statement s) {
                    return s.getString().endsWith("Smith");
                }
            });
        if (iter.hasNext()) {
            System.out.println("The database contains vcards for:");
            while (iter.hasNext()) {
                System.out.println("  " + iter.nextStatement()
                                              .getString());
            }
        } else {
            System.out.println("No Smith's were found in the database");
        }


```

### 模型操作
**Union**
融合操作，知识融合，可以用来作为扩充实体的操作
![Markdown](http://i4.bvimg.com/654958/68e8956e685f91fb.png)

```Java
// read the RDF/XML files
model1.read(new InputStreamReader(in1), "");
model2.read(new InputStreamReader(in2), "");

// merge the Models
Model model = model1.union(model2);

// print the Model as RDF/XML
model.write(system.out, "RDF/XML-ABBREV");

```
还有其他inseraction和 difference方法

## SPARSQL
导入文件 vc-db-1.rdf 文件

```text
<rdf:RDF
  xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'
  xmlns:vCard='http://www.w3.org/2001/vcard-rdf/3.0#'
   >

  <rdf:Description rdf:about="http://somewhere/JohnSmith">
    <vCard:FN>John Smith</vCard:FN>
    <vCard:N rdf:parseType="Resource">
	<vCard:Family>Smith</vCard:Family>
	<vCard:Given>John</vCard:Given>
    </vCard:N>
  </rdf:Description>
</rdf:RDF>
```
**查询 在web界面中查询**
```sql
http://192.168.130.37:3030/Demo/query
SELECT ?x
WHERE { ?x  <http://www.w3.org/2001/vcard-rdf/3.0#FN>  "John Smith" }
>>>
<http://somewhere/JohnSmith>
执行查询脚本
bin/sparql --data=doc/Tutorial/vc-db-1.rdf --query=doc/Tutorial/q1.rq


```

**查询 1 **
```sql
SELECT ?x ?fname
WHERE {?x  <http://www.w3.org/2001/vcard-rdf/3.0#FN>  ?fname}


1 <http://somewhere/JohnSmith> "John Smith" 
2 <http://somewhere/SarahJones> "Sarah Jones" 
3 <http://somewhere/MattJones> "Matt Jones" 
4 <http://somewhere/RebeccaSmith> "Becky Smith" 

```

**模式匹配式的查询**
```
SELECT ?givenName
WHERE
  { ?y  <http://www.w3.org/2001/vcard-rdf/3.0#Family>  "Smith" .
    ?y  <http://www.w3.org/2001/vcard-rdf/3.0#Given>  ?givenName .
  }

在匹配smith的情况下找出匹配的y，再根据y找出givenName
```
**Qnames**
还可以对vacrd进行查询，现将vacrd的别名写出来,然后在后面加上property等
如FAMILY 
```
PREFIX  vacrd: <http://www.w3.org/2001/vcard-rdf/3.0#>
SELECT ?givenName
WHERE
 { ?y vcard:Family "Smith" .
   ?y vcard:Given  ?givenName .
 }
```
**Blank Nodes**
如果有空的node 也是会返回的
```
PREFIX vcard:      <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?y ?givenName
WHERE
 { ?y vcard:Family "Smith" .
   ?y vcard:Given  ?givenName .
 }
```
**支持的String Matching**
```
PREFIX vcard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?g
WHERE
{ ?y vcard:Given ?g .
  FILTER regex(?g, "r", "i") }
```
i代表大小写不敏感的字符匹配模式


![Markdown](http://i4.bvimg.com/654958/911981b4d194f433.png)
看清楚详细的查询过程

然后看下面这个:
```
数据:
xmlns:info='http://somewhere/peopleInfo#'

 <rdf:Description rdf:about="http://somewhere/JohnSmith">
    <vCard:FN>John Smith</vCard:FN>
    <info:age rdf:datatype='http://www.w3.org/2001/XMLSchema#integer'>25</info:age>
    <vCard:N rdf:parseType="Resource">
	<vCard:Family>Smith</vCard:Family>
	<vCard:Given>John</vCard:Given>
    </vCard:N>
  </rdf:Description>


查询
先声明在rdf文件中的别名
PREFIX info: <http://somewhere/peopleInfo#>

SELECT ?resource   # 主
WHERE
  { # 主       谓语     宾
    ?resource info:age ?age .
    FILTER (?age >= 24)
  }

```

**OPTIONALS**
```
PREFIX info:    <http://somewhere/peopleInfo#>
PREFIX vcard:   <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name ?age
WHERE
{
    ?person vcard:FN  ?name .
    OPTIONAL { ?person info:age ?age }
}
OPTIONAL

"John Smith" "25"^^xsd:integer 
"Sarah Jones" 
"Matt Jones" 
"Becky Smith" "23"^^xsd:integer 
有的数据是没有的


```

**组合Optional 和Filter**
```
PREFIX info:    <http://somewhere/peopleInfo#>
PREFIX vcard:   <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name ?age
WHERE
{
    ?person vcard:FN  ?name .
    OPTIONAL { ?person info:age ?age.
    FILTER (?age >24)}
}
```

**占位符用[] 来表示**
**Union查询**
```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX vCard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name
WHERE
{
  {[] vCard:FN ?name}
  UNION
  {[] foaf:name ?name}
}
```

在占位符中加入过滤操作
```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX vCard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name
WHERE
{
  [] ?p ?name
  FILTER (?p = foaf:name || ?p = vCard:FN)
}


```

**UNION - remembering where the data was found**
```
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX vCard: <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?name1 ?name2
WHERE
{
   { [] foaf:name ?name1 } UNION { [] vCard:FN ?name2 }
}

---------------------------------
| name1         | name2         |
=================================
| "Matt Jones"  |               |
| "Sarah Jones" |               |
|               | "Becky Smith" |
|               | "John Smith"  |
---------------------------------
```

其实Optional的作用就类似于 || 或的感觉



## 知识图谱中的挖掘数据
