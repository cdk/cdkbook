# BackgroundColor.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.11')

import java.awt.Color;
import org.openscience.cdk.depict.*;
import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.templates.*;
import org.openscience.cdk.renderer.generators.BasicSceneGenerator;
IAtomContainer triazole = MoleculeFactory.make123Triazole()
new DepictionGenerator()
  .withSize(600, 600)
  .withMargin(0.1)
  .withZoom(3.0)
  .withAtomColors()
  .withParam(BasicSceneGenerator.BackgroundColor.class, Color.lightGray)
  .depict(triazole)
  .writeTo("BackgroundColor.png");
```
**Output:**
```plain
```
