<a name="sec:io"></a>
# Input/Output

The CDK has functionality for extracting information from files in many
different file formats. Unfortunately, hardly ever the full format
specification is supported, but generally the chemical graph and 2D or 3D
coordinates are extracted, not uncommonly complemented with formal or partial
charge.

## File Format Detection

Typically, a human is fairly aware about the format of a file, if he looks
at a file. Very often, the file extension (which is hidden on many Microsoft
Windows versions by default) gives a clear clue. Files with the .mol and .sdf
extension are very likely to have one of the MDL formats. If the file 
extension is ambiguous, a trained cheminformatician can often help you out
quickly, applying tacit knowledge about those formats.

Computer programs, however, cannot rely on file formats, and have to
formalize rules for inspecting the file content to determine what file format
it is. The CDK has such functionality available for recognizing chemical
file formats. But, to ensure no false detections are made, it
requires a fairly accurate method for detecting
the <a name="tp1">chemical format</a> of a file. Appendix [D.1](appfileformats.md#sec:fileformats)
provides a list of all chemical file formats the CDK knows about.

Programmatically, the format of a file can be detected using the
`FormatFactory`:

**<a name="script:GuessFormat">Script 11.1</a>** [code/GuessFormat.groovy](code/GuessFormat.code.md)
```groovy
Reader stringReader = new StringReader(
  "<molecule xmlns='http://www.xml-cml.org/schema'/>"
);
FormatFactory factory = new FormatFactory();
IChemFormat format = factory.guessFormat(stringReader);
System.out.println("Format: " + format.getFormatName());
```

For example, this script recognizes that a file has the Chemical Markup
Language [<a href="#citeref1">1</a>,<a href="#citeref2">2</a>] format:

```plain
Format: Chemical Markup Language
```

To learn if the CDK has a `IChemObjectReader` or
`IChemObjectWriter` for a format, one can use the methods `getReaderClassName()`
and `getWriterClassName()` respectively:

**<a name="script:HasReaderOrWriter">Script 11.2</a>** [code/HasReaderOrWriter.groovy](code/HasReaderOrWriter.code.md)
```groovy
Reader stringReader = new StringReader(
  "<molecule xmlns='http://www.xml-cml.org/schema'/>"
);
IChemFormat format = factory.guessFormat(stringReader);
String readerClass = format.getReaderClassName();
String writerClass = format.getWriterClassName();
System.out.println("Reader: " + readerClass);
System.out.println("Writer: " + writerClass);
```

It reports:

```plain
Reader: org.openscience.cdk.io.CMLReader
Writer: org.openscience.cdk.io.CMLWriter
```

### Custom format matchers

The SMILES format is one of the few formats which does not have a matcher.
This is because there is no generally accepted file format based on this
line notation.

However, we can define a custom matcher ourselves and use that. First,
the matcher will look something like:

**<a name="script:SMILESFormatMatcher">Script 11.3</a>** [code/SMILESFormatMatcher.java](code/SMILESFormatMatcher.code.md)
```java
public class SMILESFormatMatcher
  extends SMILESFormat
  implements IChemFormatMatcher {
  private static IResourceFormat instance = null;
  private SmilesParser parser = null;
  public SMILESFormatMatcher() {
    parser = new SmilesParser(
      SilentChemObjectBuilder.getInstance()
    );
  }
  public static IResourceFormat getInstance() {
    if (instance == null)
      instance = new SMILESFormatMatcher();
    return instance;
  }
  public boolean matches(int lineNumber, String line) {
    if (lineNumber == 1) {
      String[] parts = line.split(" ");
      if (parts.length == 2) {
        String smiles = parts[0];
        String name = parts[1]; // not used here
        try {
          parser.parseSmiles(smiles);
          return true;
        } catch (InvalidSmilesException exception) {}
      }
    }
    return false;
  }
  public final MatchResult matches(final List<String> lines) {
    if (lines.get(0) != null && matches(1, lines.get(0))) {
      return new MatchResult(
        true,
        (IChemFormat)SMILESFormat.getInstance(),
        Integer.valueOf(1)
      );
    }
    return new MatchResult(false, null, Integer.MAX_VALUE);
  }
}
```

If we then register this new matcher with the `FormatFactory`:

**<a name="script:GuessSMILES">Script 11.4</a>** [code/GuessSMILES.groovyl](code/GuessSMILES.code.md)
```groovy
Reader stringReader = new StringReader(
  "O=CN formamide\n" +
  "OCC ethanol\n"
);
FormatFactory factory = new FormatFactory();
factory.registerFormat(SMILESFormatMatcher.getInstance());
IChemFormat format = factory.guessFormat(stringReader);
System.out.println("Format: " + format.getFormatName());
```

And with this, we can detect a file with SMILES strings and names:

```plain
Format: SMILES
```

Keep in mind that the more specific your custom matcher is, the lower
the change of other formats accidentally recognized by your custom matcher.

REINSERT TABLE

## Reading from Readers and InputStreams

Many input readers in the CDK allow reading from a Java `Reader` class,
but all are required to also read from an `InputStream`. The difference
between these two Java classes is that the Reader is based on a character
stream, while an InputStream is based on an byte stream. For some readers this
difference is crucial: processing an XML based format, such as CML and XML
formats used by PubChem should be read from an InputStream, not a Reader.

For other formats, it does not matter. This allows, for example, to read
a file easily from a string with a `StringReader`
(mind the newlines indicated by `\n`):

**<a name="script:InputFromStringReader">Script 11.5</a>** [code/InputFromStringReader.groovy](code/InputFromStringReader.code.md)
```groovy
String bf3 = "4\n" +
"Bortrifluorid\n" +
"B    0.0000    0.0000    0.0000\n" +
"F    1.0000    0.0000    0.0000\n" +
"F   -0.5000   -0.8660    0.0000\n" +
"F   -0.5000    0.8660    0.0000\n";
reader = new XYZReader(
  new StringReader(bf3)
)
chemfile = reader.read(new ChemFile())
mol = ChemFileManipulator.getAllAtomContainers(chemfile)
  .get(0)
println "Atom count: $mol.atomCount"
```

But besides reading XML files correctly, the support for InputStream also allows
reading files directly from the internet and from gzipped files (see
Section [12.4](#sec:gzip)).

<a name="sec:domoicacid"></a>
### Example: Downloading Domoic Acid from PubChem

As an example, below will follow a small script that takes a
<a name="tp2">PubChem</a> compound
identifier (CID) and downloads the corresponding <a name="tp3">ASN.1</a> XML file, parses it and
counts the number of atoms:

**<a name="script:PubChemDownload">Script 11.6</a>** [code/PubChemDownload.groovy](code/PubChemDownload.code.md)
```groovy
cid = 5282253
reader = new PCCompoundXMLReader(
  new URL(
    "https://pubchem.ncbi.nlm.nih.gov/summary/" +
    "summary.cgi?cid=$cid&disopt=SaveXML"
  ).newInputStream()
)
mol = reader.read(new AtomContainer())
println "CID: " + mol.getProperty("PubChem CID")
println "Atom count: $mol.atomCount"
```

It reports:

```plain
CID: 5282253
Atom count: 43
```

PubChem ASN.1 files come with an extensive list of molecular properties. These
are stored as properties on the molecule object and can be retrieved using the
`getProperties()` method, or, using the Groovy bean formalism:

**<a name="script:PubChemDownloadProperties">Script 11.7</a>** [code/PubChemDownloadProperties.groovy](code/PubChemDownloadProperties.code.md)
```groovy
mol.properties.each {
  line = "" + it
  println line
}
```

which lists the properties for the earlier downloaded domoic acid:

```plain
PubChem CID=5282253
Compound Complexity=510
Fingerprint (SubStructure Keys)=00000371E0723800...
  0000000000000000000000000001600000000000000000...
  00000000000000001E00100800000D28C18004020802C0...
  0200880220D208000000002000000808818800080A0012...
  00812004400004D000988003BC7F020E80000000000000...
  000000000000000000000000000000
IUPAC Name (Allowed)=(2S,3S,4S)-3-(carboxymethyl...
  )-4-[(1Z,3E,5R)-5-carboxy-1-methyl-hexa-1,3-di...
  enyl]pyrrolidine-2-carboxylic acid
IUPAC Name (CAS-like Style)=(2S,3S,4S)-4-[(2Z,4E...
  ,6R)-6-carboxyhepta-2,4-dien-2-yl]-3-(carboxym...
  ethyl)-2-pyrrolidinecarboxylic acid
IUPAC Name (Markup)=(2<I>S</I>,3<I>S</I>,4<I>S</...
  I>)-4-[(2<I>Z</I>,4<I>E</I>,6<I>R</I>)-6-carbo...
  xyhepta-2,4-dien-2-yl]-3-(carboxymethyl)pyrrol...
  idine-2-carboxylic acid
IUPAC Name (Preferred)=(2S,3S,4S)-4-[(2Z,4E,6R)-...
  6-carboxyhepta-2,4-dien-2-yl]-3-(carboxymethyl...
  )pyrrolidine-2-carboxylic acid
IUPAC Name (Systematic)=(2S,3S,4S)-3-(2-hydroxy-...
  2-oxoethyl)-4-[(2Z,4E,6R)-6-methyl-7-oxidanyl-...
  7-oxidanylidene-hepta-2,4-dien-2-yl]pyrrolidin...
  e-2-carboxylic acid
IUPAC Name (Traditional)=(2S,3S,4S)-3-(carboxyme...
  thyl)-4-[(1Z,3E,5R)-5-carboxy-1-methyl-hexa-1,...
  3-dienyl]proline
InChI (Standard)=InChI=1S/C15H21NO6/c1-8(4-3-5-9...
  (2)14(19)20)11-7-16-13(15(21)22)10(11)6-12(17)...
  18/h3-5,9-11,13,16H,6-7H2,1-2H3,(H,17,18)(H,19...
  ,20)(H,21,22)/b5-3+,8-4-/t9-,10+,11-,13+/m1/s1
InChIKey (Standard)=VZFRNCSOCOPNDB-AOKDLOFSSA-N
Log P (XLogP3-AA)=-1.3
Mass (Exact)=311.13688739
Molecular Formula=C15H21NO6
Molecular Weight=311.33
SMILES (Canonical)=CC(C=CC=C(C)C1CNC(C1CC(=O)O)C...
  (=O)O)C(=O)O
SMILES (Isomeric)=C[C@H](/C=C/C=C(/C)\textbacksl...
  ash[C@H]1CN[C@@H]([C@H]1CC(=O)O)C(=O)O)C(=O)O
Topological (Polar Surface Area)=124
Weight (MonoIsotopic)=311.13688739
```


## Input Validation

The history of the CDK project has seen many bug reports about problems
which in fact turned out to be problems with in the input file. While
the general perception seems to be that because files could be written,
the content must be consistent.

However, this is a strong misconception. There are several problems
found in chemical files in the wild. A first common problem is that the
file is not conform the syntax of the specification. An example here
can be that at places where a number is expected, something else is
given; not uncommonly, this is caused by incorrect use of whitespace.

A second problem is that the file looks perfectly reasonable, but that
the software that wrote the file used conventions and extensions that
are not supported by the reading software. A common example is the use
of the D and T symbols, for deuterium and tritium in MDL molfiles,
where the specification does not allow that.

A third problem is that most chemical file formats do not disallow
incorrect chemical graphs. For example, formats often allow to
bind an atom to itself, which will cause problems when analyzing
this graph. These problems are much more rare, though.

<a name="sec:readingModes"></a>
### Reading modes

The [`IChemObjectReader`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/io/IChemObjectReader.html) has a feature that allows setting
a validating mode, which has two values:

**<a name="script:ReadingModes">Script 11.8</a>** [code/ReadingModes.groovy](code/ReadingModes.code.md)
```groovy
IChemObjectReader.Mode.each {
  println it
}
```

returning:

```plain
RELAXED
STRICT
```

The `STRICT` mode follows the exact format specification. There
`RELAXED` mode allows for a few common extensions, such as
the support for the T and D element types. For example, let's consider
this file:

```plain

  CDK

  3  2  0  0  0  0  0  0  0  0999 V2000
    2.5369   -0.1550    0.0000 O   0  0  0  0  0  0  0  0  0  0  0  0
    3.0739    0.1550    0.0000 D   1  0  0  0  0  0  0  0  0  0  0  0
    2.0000    0.1550    0.0000 T   1  0  0  0  0  0  0  0  0  0  0  0
  1  2  1  0  0  0  0
  1  3  1  0  0  0  0
M  ISO   2   2   2   3   3
M  END
```

If we read this file with:

**<a name="script:ReadStrict">Script 11.9</a>** [code/ReadStrict.groovy](code/ReadStrict.code.md)
```groovy
reader = new MDLV2000Reader(
  new File("data/t.mol").newReader(),
  Mode.STRICT
);
water = reader.read(new AtomContainer());
println "atom count: $water.atomCount"
```

we get this exception:

```plain
invalid symbol: D
```

However, if we read the file in `RELAXED` mode with this code:

**<a name="script:ReadRelaxed">Script 11.10</a>** [code/ReadRelaxed.groovy](code/ReadRelaxed.code.md)
```groovy
reader = new MDLV2000Reader(
  new File("data/t.mol").newReader(),
  Mode.RELAXED
);
water = reader.read(new AtomContainer());
println "atom count: $water.atomCount"
```

the files will be read as desired:

```plain
atom count: 3
```

### Validation

When a file is being read in `RELAXED` mode, it is possible to get
error messages. This functionality is provided by the
[`IChemObjectReaderErrorHandler`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/io/IChemObjectReaderErrorHandler.html) support in
[`IChemObjectReader`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/io/IChemObjectReader.html).
For example, we can define this custom error handler:

**<a name="script:CustomErrorHandler">Script 11.11</a>** [code/CustomErrorHandler.groovy](code/CustomErrorHandler.code.md)
```groovy
class ErrorHandler
implements IChemObjectReaderErrorHandler {
  public void handleError(String message) {
    println message;
  };
  public void handleError(String message,
    Exception exception)
  {
    println message + "\n  -> " +
            exception.getMessage();
  };
  public void handleError(String message,
    int row, int colStart, int colEnd)
  {
    print "location: " + row + ", " + 
          colStart + "-" + colEnd + ": ";
    println message;
  };
  public void handleError(String message,
    int row, int colStart, int colEnd,
    Exception exception)
  {
    print "location: " + row + ", " +
          colStart + "-" + colEnd + ": "
    println message + "\n  -> " +
            exception.getMessage()
  };
}
```

and use that when reading a file:

**<a name="script:ReadErrorHandler">Script 11.12</a>** [code/ReadErrorHandler.groovy](code/ReadErrorHandler.code.md)
```groovy
reader = new MDLV2000Reader(
  new File("data/t.mol").newReader(),
  Mode.RELAXED
);
reader.setErrorHandler(new ErrorHandler());
water = reader.read(new AtomContainer());
```

we get these warnings via the handler interface:

```plain
location: 6, 31-33: invalid symbol: D
location: 7, 31-33: invalid symbol: T
```

Because of an issue in version 2.8 of the CDK, the above does not show any warnings.
This has been fixed in CDK 2.3, see [commit 547b028e17656f54a080a885a166377320b3a8ad](https://github.com/cdk/cdk/commit/547b028e17656f54a080a885a166377320b3a8ad).

<a name="sec:gzip"></a>
## Gzipped files

Some remote databases <a name="tp4">gzip</a> their data files to reduce download sized.
The Protein Brookhaven Database (<a name="tp5">PDB</a>) is such a database. Fortunately, Java
has a simple API to work with gzipped files, using the `GZIPInputStream`:

**<a name="script:PDBCoordinateExtraction">Script 11.13</a>** [code/PDBCoordinateExtraction.groovy](code/PDBCoordinateExtraction.code.md)
```groovy
reader = new PDBReader(
  new GZIPInputStream(
    new URL(
      "http://files.rcsb.org/download/1CRN.pdb.gz"
    ).openStream()
  )
);
crambin = reader.read(new ChemFile());
for (container in
     ChemFileManipulator.getAllAtomContainers(
       crambin
     )) {
  for (atom in container.atoms()) {
    println atom.point3d;
  }
}
```

## Iterating Readers

By default, the CDK readers read structures into memory. This is fine when it
is a relatively small model. It no longer works for large files, such as 1GB
<a name="tp6">MDL SD files</a> [<a href="#citeref3">3</a>]. To allow processing of
such large files, the CDK can take
advantage from the fact that these SD files are basically a concatenation of
MDL molfiles. Therefore, one can use an iterating reader to process each
individual molecule one by one.

<a name="sec:sdfiles"></a>
### MDL SD files

MDL SD files can be processed using the [`IteratingSDFReader`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/io/iterator/IteratingSDFReader.html), for
example, to generate a SMILES for each structure:

**<a name="script:IteratingSDFReaderDemo">Script 11.14</a>** [code/IteratingSDFReaderDemo.groovy](code/IteratingSDFReaderDemo.code.md)
```groovy
iterator = new IteratingSDFReader(
  new File("data/test6.sdf").newReader(),
  DefaultChemObjectBuilder.getInstance()
)
while (iterator.hasNext()) {
  IAtomContainer mol = iterator.next()
  formula = MolecularFormulaManipulator
    .getMolecularFormula(mol)
  println MolecularFormulaManipulator
    .getString(formula)
}
```

Which outputs the molecular formula for the three entries in the file:

```plain
C19H24Br2N2O6
C20H24N2O5S
C17H22N2O6S
```

<a name="sec:pubchemfiles"></a>
### PubChem Compounds XML files

Similarly, PubChem Compounds XML files can be processed taking advantage
of a XML pull library, which is nicely hidden behind the same iterator
interface as used for parsing MDL SD files. Iterating over a set
of compounds is fairly straightforward with the
[`IteratingPCCompoundXMLReader`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/io/iterator/IteratingPCCompoundXMLReader.html) class:

**<a name="script:PubChemCompoundsXMLDemo">Script 11.15</a>** [code/PubChemCompoundsXMLDemo.groovy](code/PubChemCompoundsXMLDemo.code.md)
```groovy
iterator = new IteratingPCCompoundXMLReader(
  new File("data/aceticAcids38.xml").newReader(),
  DefaultChemObjectBuilder.getInstance()
)
while (iterator.hasNext()) {
  IAtomContainer mol = iterator.next()
  formula = MolecularFormulaManipulator
    .getMolecularFormula(mol)
  println MolecularFormulaManipulator.getString(formula)
}
```

Which outputs the molecular formula for the three entries in the
`aceticAcids38.xml` file:

```plain
C2H4O2
[C2H3O2]-
[C2H3HgO2]+
```


## Customizing the Output

An interesting feature of file IO in the CDK is that it is customizable. Before
I will give all the details, let's start with a simple example: creating a
<a name="tp7">Gaussian input file</a> for optimizing the structure of methane,
and let's start with an XYZ file, that is, with `methane.xyz`:

```
5
methane
C  0.25700 -0.36300  0.00000
H  0.25700  0.72700  0.00000
H  0.77100 -0.72700  0.89000
H  0.77100 -0.72700 -0.89000
H -0.77100 -0.72700  0.00000
```

The output will look something like:

```
%nprocl=5
# b3lyp/6-31g opt

Job started on Linux cluster on 20041010.

0 1
C 0 0.257 -0.363 0.0
H 0 0.257 0.727 0.0
H 0 0.771 -0.727 0.89
H 0 0.771 -0.727 -0.89
H 0 -0.771 -0.727 0.0

```

The writer used the default IO options in the above example. So, the next step is to see
which options the writer allows. To get a list of options for a certain IO
class in one does something along the lines:

**<a name="script:ListIOOptions">Script 11.16</a>** [code/ListIOOptions.groovy](code/ListIOOptions.code.md)
```groovy
IChemObjectWriter writer = new GaussianInputWriter();
for (IOSetting setting : writer.getIOSettings()) {
  println "[" + setting.getName() + "]"
  println "Option: " + setting.getQuestion()
  println "Current value: " + setting.getSetting()
}
```

which results in the following output:

```plain
[OpenShell]
Option: Should the calculation be open shell?
Current value: false
[Comment]
Option: What comment should be put in the file?
Current value: Created with CDK (http://cdk.sf.n...
  et/)
[Memory]
Option: How much memory do you want to use?
Current value: unset
[Command]
Option: What kind of job do you want to perform?
Current value: energy calculation
[ProcessorCount]
Option: How many processors should be used by Ga...
  ussian?
Current value: 1
```

### Setting Properties

The IO settings system allows interactive setting of these options, but a
perfectly fine alternative is to use a Java Properties object.

Consider the following source code:

**<a name="script:PropertiesSettings">Script 11.17</a>** [code/PropertiesSettings.groovy](code/PropertiesSettings.code.md)
```groovy
// the custom settings
Properties customSettings = new Properties();
customSettings.setProperty("Basis",   "6-31g*");
customSettings.setProperty("Command",
  "geometry optimization");
customSettings.setProperty("Comment",
  "Job started on Linux cluster on 20041010.");
customSettings.setProperty("ProcessorCount", "5");
PropertiesListener listener = new PropertiesListener(
  customSettings
);
// create the writer
GaussianInputWriter writer = new GaussianInputWriter(
  new FileWriter(new File("methane.gin"))
);
writer.addChemObjectIOListener(listener);
XYZReader reader = new XYZReader(
  new FileReader(new File("data/methane.xyz"))
);
// convert the file
ChemFile content = (ChemFile)reader.read(new ChemFile());
IAtomContainer molecule = content.getChemSequence(0).
  getChemModel(0).getMoleculeSet().getAtomContainer(0);
writer.write(molecule);
writer.close();
```

The `PropertiesListener` takes a `Properties` class as parameter in
its constructor. Therefore, the properties are defined by the
`customSettings` variable in the first few lines. The
`PropertiesListener` `listener` is the instantiated with the
customizations as constructor parameter.

The output writer, specified to write to the `methane.gin` file, is
created after which the `ChemObjectIOListener` is set. Only by setting
this listener, the output will be customized with the earlier defined
properties. The rest of the code reads a molecule from an XYZ file and writes
the `content` to the created Gaussian Input file.

## Example: creating unit tests for atom type perception

We saw earlier an example for reading files directly from PubChem
(see Section [12.2.1](#sec:domoicacid)).
This can be conveniently used to create `CDK source code`, for example,
for use in unit tests for the atom type perception code (see
Section [13.2](atomtype.md#sec:atomtypePerception)). But because we do not want
2D and 3D coordinates being set in the source code, we disable those
options:

**<a name="script:AtomTypeUnitTest">Script 11.18</a>** [code/AtomTypeUnitTest.groovy](code/AtomTypeUnitTest.code.md)
```groovy
cid = 3396560
mol = reader.read(new AtomContainer())
stringWriter = new StringWriter();
CDKSourceCodeWriter writer =
  new CDKSourceCodeWriter(stringWriter);
customSettings = new Properties();
customSettings.setProperty("write2DCoordinates", "false");
customSettings.setProperty("write3DCoordinates", "false");
writer.addChemObjectIOListener(
  new PropertiesListener(
    customSettings
  )
)
writer.write(mol);
writer.close();
println stringWriter.toString();
```

This results in this source code:

```plain
{
  IChemObjectBuilder builder = DefaultChemObject...
  Builder.getInstance();
  IAtomContainer mol = builder.newInstance(IAtom...
  Container.class);
  IAtom a1 = builder.newInstance(IAtom.class,"P");
  a1.setFormalCharge(0);
  mol.addAtom(a1);
  IAtom a2 = builder.newInstance(IAtom.class,"O");
  a2.setFormalCharge(0);
  mol.addAtom(a2);
  IAtom a3 = builder.newInstance(IAtom.class,"O");
  a3.setFormalCharge(0);
  mol.addAtom(a3);
  IAtom a4 = builder.newInstance(IAtom.class,"C");
  a4.setFormalCharge(0);
  mol.addAtom(a4);
  IAtom a5 = builder.newInstance(IAtom.class,"H");
  a5.setFormalCharge(0);
  mol.addAtom(a5);
  IAtom a6 = builder.newInstance(IAtom.class,"H");
  a6.setFormalCharge(0);
  mol.addAtom(a6);
  IAtom a7 = builder.newInstance(IAtom.class,"H");
  a7.setFormalCharge(0);
  mol.addAtom(a7);
  IAtom a8 = builder.newInstance(IAtom.class,"H");
  a8.setFormalCharge(0);
  mol.addAtom(a8);
  IAtom a9 = builder.newInstance(IAtom.class,"H");
  a9.setFormalCharge(0);
  mol.addAtom(a9);
  IBond b1 = builder.newInstance(IBond.class,a1,...
   a2, IBond.Order.SINGLE);
  mol.addBond(b1);
  IBond b2 = builder.newInstance(IBond.class,a1,...
   a3, IBond.Order.DOUBLE);
  mol.addBond(b2);
  IBond b3 = builder.newInstance(IBond.class,a1,...
   a4, IBond.Order.SINGLE);
  mol.addBond(b3);
  IBond b4 = builder.newInstance(IBond.class,a1,...
   a5, IBond.Order.SINGLE);
  mol.addBond(b4);
  IBond b5 = builder.newInstance(IBond.class,a2,...
   a9, IBond.Order.SINGLE);
  mol.addBond(b5);
  IBond b6 = builder.newInstance(IBond.class,a4,...
   a6, IBond.Order.SINGLE);
  mol.addBond(b6);
  IBond b7 = builder.newInstance(IBond.class,a4,...
   a7, IBond.Order.SINGLE);
  mol.addBond(b7);
  IBond b8 = builder.newInstance(IBond.class,a4,...
   a8, IBond.Order.SINGLE);
  mol.addBond(b8);
}

```

<a name="sec:lineNotations"></a>
## Line Notations

Another common input mechanism in cheminformatics is the <a name="tp8">line notation</a>.
Several line notations have been proposed, including the <a name="tp9">Wiswesser Line Notation</a>
(WLN) [<a href="#citeref4">4</a>] and the <a name="tp10">Sybyl Line Notation</a> (SLN) [<a href="#citeref5">5</a>],
but the most popular is <a name="tp11">SMILES</a> [<a href="#citeref6">6</a>]. There is a Open Standard around
this format called <a name="tp12">OpenSMILES</a>, available at [http://www.opensmiles.org/](http://www.opensmiles.org/).

### SMILES

The CDK can both read and write SMILES, or at least a significant subset of the
line notation. You can parse a SMILES into a IAtomContainer with the
[`SmilesParser`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/smiles/SmilesParser.html). The constructor of the parser takes an [`IChemObjectBuilder`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/interfaces/IChemObjectBuilder.html) (see Section [11](builders.md#sec:builders))
because it needs to know what CDK interface implementation it must use to create
classes. This example uses the [`DefaultChemObjectBuilder`](http://cdk.github.io/cdk/latest/docs/api/org/openscience/cdk/DefaultChemObjectBuilder.html):

**<a name="script:ReadSMILES">Script 11.19</a>** [code/ReadSMILES.groovy](code/ReadSMILES.code.md)
```groovy
sp = new SmilesParser(
  DefaultChemObjectBuilder.getInstance()
)
mol = sp.parseSmiles("CC(=O)OC1=CC=CC=C1C(=O)O")
println "Aspirin has ${mol.atomCount} atoms."
```

Telling us the number of (non-hydrogen) atoms in aspirin:

```plain
Aspirin has 13 atoms.
```

Writing of SMILES goes in a similar way. But I do like to point out that by default
the `SMILESGenerator` does not use the convention to use lower case element
symbols for aromatic atoms. To trigger that, you should use the
`setUseAromaticityFlag` method:

**<a name="script:WriteSMILES">Script 11.20</a>** [code/WriteSMILES.groovy](code/WriteSMILES.code.md)
```groovy
mol = MoleculeFactory.makePhenylAmine()
generator = SmilesGenerator.generic()
smiles = generator.createSMILES(mol)
println "Ph-NH2 -> $smiles"
generator = SmilesGenerator.generic().aromatic()
smiles = generator.createSMILES(mol)
println "Ph-NH2 -> $smiles"
```

showing the different output without and with that option set:

```plain
Ph-NH2 -> C1(=CC=CC=C1)N
Ph-NH2 -> c1(ccccc1)N
```

Of course, this does require that aromaticity has been perceived, as explained
in Section [17.5](properties.md#sec:aromaticity).

## Recipes

This section will list for a few formats a recipe for how to read content from those
formats, taking into account common issues with the input.

### MDL molfile (V2000)

Like any file format, they support a limited number of features. For example,
MDL files cannot represent a bond order 4, a quadruple bond.
Other missing explicit details include hydrogens, and atom-based stereochemistry.
Stereochemistry is wedge-bond-based, see Section ??.

An example file which uses the bond order 4, is this file:

```plain

  CDK

 10 11  0  0  0  0  0  0  0  0999 V2000
  208.0000  866.5142    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  175.5651  882.1340    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  167.5544  917.2314    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  190.0000  945.3774    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  226.0000  945.3774    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  248.4456  917.2314    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  240.4349  882.1340    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  271.3391  863.6697    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  298.4496  887.3555    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  284.3007  920.4585    0.0000 C   0  0  0  0  0  0  0  0  0  0  0  0
  1  2  4  0  0  0  0 
  2  3  4  0  0  0  0 
  3  4  4  0  0  0  0 
  4  5  4  0  0  0  0 
  5  6  4  0  0  0  0 
  6  7  4  0  0  0  0 
  7  1  4  0  0  0  0 
  7  8  4  0  0  0  0 
  8  9  4  0  0  0  0 
  9 10  4  0  0  0  0 
 10  6  4  0  0  0  0 
M  END
```

More recent MDL formats have become more powerful. The V3000 format can do
much more than the V2000 format, or even the pre-V2000 format.

Here's a recipe with inline comments:

**<a name="script:InputMDLMolfiles">Script 11.21</a>** [code/InputMDLMolfiles.groovy](code/InputMDLMolfiles.code.md)
```groovy
reader = new MDLV2000Reader(
  new File("data/azulene4.mol").newReader(),
  Mode.RELAXED
);
azulene = reader.read(new AtomContainer());
// perceive atom types
AtomContainerManipulator
  .percieveAtomTypesAndConfigureAtoms(
  azulene
)
// add missing hydrogens
adder.addImplicitHydrogens(azulene);
// if bond order 4 was present,
// deduce bond orders
Kekulization.kekulize(azulene);
println "Atom count: " + azulene.atomCount
doubleBondCount = 0
singleBondCount = 0
for (bond in azulene.bonds()) {
  if (bond.order == Order.DOUBLE)
    doubleBondCount++
  if (bond.order == Order.SINGLE) 
    singleBondCount++
}
println "Single bonds: " + singleBondCount
println "Double bonds: " + doubleBondCount
```

This code will perceive CDK atom types. These types are needed to add the missing
hydrogens, as well as to resolve the bond order information. The input has ten atoms
and eleven bonds, all marked with bond order 4.

The result of the above post-processing is:

```plain
Atom count: 10
Single bonds: 6
Double bonds: 5
```


## References

1. <a name="citeref1"></a>Murray-Rust P, Rzepa HS. Chemical Markup, XML, and the Worldwide Web. 1. Basic Principles. JCICS. 1999 Nov;39(6):928–42.  doi:[10.1021/CI990052B](https://doi.org/10.1021/CI990052B) ([Scholia](https://scholia.toolforge.org/doi/10.1021/CI990052B))
2. <a name="citeref2"></a>Willighagen E. Processing CML conventions in Java. Internet Journal of Chemistry [Internet]. 2001 Feb 12;4:4. Available from: https://zenodo.org/record/1495470 doi:[10.5281/ZENODO.1495470](https://doi.org/10.5281/ZENODO.1495470) ([Scholia](https://scholia.toolforge.org/doi/10.5281/ZENODO.1495470))
3. <a name="citeref3"></a>Dalby A, Nourse JG, Hounshell WD, Gushurst AKI, Grier DL, Leland BA, et al. Description of several chemical structure file formats used by computer programs developed at Molecular Design Limited. JCICS. 1992 May 1;32(3):244–55.  doi:[10.1021/CI00007A012](https://doi.org/10.1021/CI00007A012) ([Scholia](https://scholia.toolforge.org/doi/10.1021/CI00007A012))
4. <a name="citeref4"></a>Wiswesser WJ. How the WLN began in 1949 and how it might be in 1999. JCICS. 1982 May 1;22(2):88–93.  doi:[10.1021/CI00034A005](https://doi.org/10.1021/CI00034A005) ([Scholia](https://scholia.toolforge.org/doi/10.1021/CI00034A005))
5. <a name="citeref5"></a>Homer RW, Swanson J, Jilek RJ, Hurst T, Clark RD. SYBYL line notation (SLN): a single notation to represent chemical structures, queries, reactions, and virtual libraries. JCIM. 2008 Dec 1;48(12):2294–307.  doi:[10.1021/CI7004687](https://doi.org/10.1021/CI7004687) ([Scholia](https://scholia.toolforge.org/doi/10.1021/CI7004687))
6. <a name="citeref6"></a>Weininger D. SMILES, a chemical language and information system. 1. Introduction to methodology and encoding rules. JCICS [Internet]. 1988 Feb 1;28(1):31–6. Available from: http://organica1.org/seminario/weininger88.pdf doi:[10.1021/CI00057A005](https://doi.org/10.1021/CI00057A005) ([Scholia](https://scholia.toolforge.org/doi/10.1021/CI00057A005))

