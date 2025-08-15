# RebondToolDemo.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.11')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;
import org.openscience.cdk.config.*;
import org.openscience.cdk.graph.rebond.*;
import javax.vecmath.Point3d;

methane = new AtomContainer();
methane.addAtom(new Atom("C", new Point3d(0.0, 0.0, 0.0)));
methane.addAtom(new Atom("H", new Point3d(0.6, 0.6, 0.6)));
methane.addAtom(new Atom("H", new Point3d(-0.6, -0.6, 0.6)));
methane.addAtom(new Atom("H", new Point3d(0.6, -0.6, -0.6)));
methane.addAtom(new Atom("H", new Point3d(-0.6, 0.6, -0.6)));

factory = AtomTypeFactory.getInstance(
  "org/openscience/cdk/config/data/jmol_atomtypes.txt", 
  methane.getBuilder()
);
for (IAtom atom : methane.atoms()) {
  factory.configure(atom);
}
RebondTool rebonder = new RebondTool(2.0, 0.5, 0.5);
rebonder.rebond(methane);
println "Bond count: $methane.bondCount"
```
**Output:**
```plain
Bond count: 4
```
