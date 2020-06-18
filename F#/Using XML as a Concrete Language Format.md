# Using XML as a Concrete Language Format

Two common sources of structured data in a textual format are data formatted in the Extensible Markup Language (XML) and in JavaScript Object Notation (JSON). F# comes with well-engineered libraries for reading and generating both XML and JSON, and in both cases you can manipulate a typed abstract representation of an XML or JSON document in memory, without having to worry about parsing or generating strings corresponding to the concrete format. Let’s first look at using XML data.

## Using the System.Xml Namespace

XML is a general-purpose markup language; it is extensible, because it allows its users to define their own tags. Its primary purpose is to facilitate the sharing of data across different information systems, particularly via the Internet. Here is a sample fragment of XML, defined as a string directly in F#:

```F#
let inp = """<?xml version="1.0" encoding="utf-8" ?>
<Scene>
<Composite>
<Circle radius='2' x='1' y='0'/>
<Composite>
<Circle radius='2' x='4' y='0'/>
<Square side='2' left='-3' top='0'/>
</Composite>
<Ellipse top='2' left='-2' width='3' height='4'/>
</Composite>
</Scene>"""
```

The backbone of an XML document is a hierarchical structure, and each node is decorated with attributes keyed by name. You can parse XML using the types and methods in the System.Xml namespace provided by the .NET libraries and then examine the structure of the XML interactively:

```F#
> open System.Xml;;

> let doc = new XmlDocument();;
val doc : XmlDocument

> doc.LoadXml(inp);;
val it : unit = ()

> doc.ChildNodes;;
val it : XmlNodeList =
  seq [seq []; seq [seq [seq []; seq [seq []; seq []]; seq []]]]
```

The default F# Interactive display for the `XmlNode` type isn’t particularly useful. Luckily, you can add an interactive printer to the fsi.exe session using the AddPrinter method on the fsi object:

```F#
> fsi.AddPrinter(fun (x:XmlNode) -> x.OuterXml);;

> doc.ChildNodes;;
val it : XmlNodeList =
  seq
    [<?xml version="1.0" encoding="utf-8"?>;
     <Scene><Composite><Circle radius="2" x="1" y="0" /><Composite>...</Scene>]

> doc.ChildNodes.Item(1);;
val it : XmlNode =
  <Scene><Composite><Circle radius="2" x="1" y="0" /><Composite>...</Scene>

> doc.ChildNodes.Item(1).ChildNodes.Item(0);;
val it : XmlNode =
  <Composite><Circle radius="2" x="1" y="0" />...</Composite>

> doc.ChildNodes.Item(1).ChildNodes.Item(0).ChildNodes.Item(0);;
val it : XmlNode = <Circle radius="2" x="1" y="0" />

> doc.ChildNodes.Item(1).ChildNodes.Item(0).ChildNodes.Item(0).Attributes;;
val it : XmlAttributeCollection = seq [radius="2"; x="1"; y="0"]
```

Table 8-7 shows the most commonly used types and members from the `System.Xml` namespace.

Table 8-7. Commonly Used Types and Members from the `System.Xml` Namespace

Type/Member Description

type XmlNode Represents a single node in an XML document
    member ChildNodes Gets all the child nodes of an XmlNode
    member Attributes Gets all the attributes of an XmlNode
    member OuterXml Gets the XML text representing the node and all its children
    member InnerText Gets the concatenated values of the node and all its children
    member SelectNodes Selects child nodes using an XPath query

type XmlElement Represents an element in an XML document; also an XmlNode

type XmlAttribute Represents one attribute for an XmlNode; also an XmlNode
    member Value Gets the string value for the attribute

type XmlDocument Represents an entire XML document; also an XmlNode
    member Load Populates the document from the given XmlReader, stream, or file name
    member LoadXml Populates the document object from the given XML string

type XmlReader Represents a reader for an XML document or source

type XmlWriter Represents a writer for an XML document

## From Concrete XML to Abstract Syntax

Often, your first task in processing a concrete language is to bring the language fragments under the type discipline of F#. This section shows how to transform the data contained in the XML from the previous section into an instance of the recursive type shown here. This kind of type is usually called an abstract syntax tree (AST):

```F#
open System.Drawing
type Scene =
    | Ellipse of RectangleF
    | Rect of RectangleF
    | Composite of Scene list
```

This example uses the types `PointF` and `RectangleF` from the `System.Drawing` namespace, although you can also define your own types to capture the information carried by the leaves of the tree. Listing 8-1 shows a recursive transformation that converts XML documents like the one used in the previous section into the type Scene.

Listing 8-1. Converting XML into a typed format using the System.Xml namespace

```F#
open System.Xml
open System.Drawing

type Scene =
    | Ellipse of RectangleF
    | Rect of RectangleF
    | Composite of Scene list

    /// A derived constructor
    static member Circle(center : PointF, radius) =
        Ellipse(RectangleF(center.X - radius, center.Y - radius,
                    radius * 2.0f, radius * 2.0f))

    /// A derived constructor
    static member Square(left, top, side) =
        Rect(RectangleF(left, top, side, side))

/// Extract a number from an XML attribute collection
let extractFloat32 attrName (attribs : XmlAttributeCollection) =
    float32 (attribs.GetNamedItem(attrName).Value)

/// Extract a Point from an XML attribute collection
let extractPointF (attribs : XmlAttributeCollection) =
    PointF(extractFloat32 "x" attribs, extractFloat32 "y" attribs)

/// Extract a Rectangle from an XML attribute collection
let extractRectangleF (attribs : XmlAttributeCollection) =
    RectangleF(
        extractFloat32 "left" attribs, extractFloat32 "top" attribs,
        extractFloat32 "width" attribs, extractFloat32 "height" attribs)

/// Extract a Scene from an XML node
let rec extractScene (node : XmlNode) =
    let attribs = node.Attributes
    let childNodes = node.ChildNodes
    match node.Name with
    | "Circle"  ->
        Scene.Circle(extractPointF(attribs), extractFloat32 "radius" attribs)
    | "Ellipse"  ->
        Scene.Ellipse(extractRectangleF(attribs))
    | "Rectangle"  ->
        Scene.Rect(extractRectangleF(attribs))
    | "Square"  ->
        Scene.Square(extractFloat32 "left" attribs, extractFloat32 "top" attribs,
            extractFloat32 "side" attribs)
    | "Composite"   ->
        Scene.Composite [for child in childNodes -> extractScene(child)]
    | _ -> failwithf "unable to convert XML '%s'" node.OuterXml

/// Extract a list of Scenes from an XML document
let extractScenes (doc : XmlDocument) =
   [for node in doc.ChildNodes do
        if node.Name = "Scene" then
            yield (Composite
                [for child in node.ChildNodes -> extractScene(child)])]
```

The inferred types of these functions are:

```F#
type Scene =
  | Ellipse of RectangleF
  | Rect of RectangleF
  | Composite of Scene list
  static member Circle : center:PointF * radius:float32 -> Scene
  static member Square : left:float32 * top:float32 * side:float32 -> Scene

val extractFloat32 : attrName:string -> attribs:XmlAttributeCollection -> float32
val extractPointF : attribs:XmlAttributeCollection -> PointF
val extractRectangleF : attribs:XmlAttributeCollection -> RectangleF
val extractScene : node:XmlNode -> Scene
val extractScenes : doc:XmlDocument -> Scene list
```

The definition of extractScenes in Listing 8-1 generates lists using sequence expressions, which are covered in Chapter 3. You can now apply the extractScenes function to the original XML. (You must first add a pretty-printer to the F# Interactive session for the RectangleF type using the AddPrinter function on the fsi object.)

```F#
> fsi.AddPrinter(fun (r:RectangleF) ->
    sprintf "[%A,%A,%A,%A]" r.Left r.Top r.Width r.Height);;
> extractScenes doc;;
val it : Scene list
= [Composite
    [Composite
      [Ellipse [-1.0f,-2.0f,4.0f,4.0f];
Composite [Ellipse [2.0f,-2.0f,4.0f,4.0f]; Rect [-3.0f,0.0f,2.0f,2.0f]];
Ellipse [-2.0f,2.0f,3.0f,4.0f]]]]
```

The following sections more closely explain some of the choices we’ve made in the abstract syntax design for the type Scene.

■ Tip 

translating to a typed representation isn’t always necessary. Some manipulations and analyses are better performed directly on heterogeneous, general-purpose formats, such as XML or even strings. For example, xMl libraries support xpath, accessed via the SelectNodes method on the XmlNode type. if you need to query a large, semi-structured document whose schema is frequently changing in minor ways, using xpath is the right way to do it. likewise, if you need to write a significant amount of code that interprets or analyzes a tree structure, converting to a typed abstract syntax tree is usually better.