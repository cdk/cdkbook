# DistanceMatrix.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.8')

import org.openscience.cdk.*;
import org.openscience.cdk.config.*;
import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.io.*;
import org.openscience.cdk.graph.matrix.*;
import org.openscience.cdk.tools.manipulator.*;
import org.openscience.cdk.graph.rebond.*;
import java.io.File;

reader = new XYZReader(
  new File("data/ethanoicacid.xyz").newReader()
);
allContent = ChemFileManipulator.getAllAtomContainers(
  reader.read(new ChemFile())
)
ethanoicAcid = allContent.get(0)

factory = AtomTypeFactory.getInstance(
  "org/openscience/cdk/config/data/jmol_atomtypes.txt", 
  ethanoicAcid.getBuilder()
);
for (IAtom atom : ethanoicAcid.atoms()) {
  factory.configure(atom);
}
RebondTool rebonder = new RebondTool(2.0, 0.5, 0.5);
rebonder.rebond(ethanoicAcid);

int[][] matrix = TopologicalMatrix.getMatrix(ethanoicAcid)
for (row=0;row<ethanoicAcid.getAtomCount();row++) {
  for (col=0;col<ethanoicAcid.getAtomCount();col++) {
    print matrix[row][col] + " "
  }
  println ""
}
```
**Output:**
```plain
0 1 2 2 
1 0 1 1 
2 1 0 2 
2 1 2 0 
```
