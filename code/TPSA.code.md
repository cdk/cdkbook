# TPSA.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

import org.openscience.cdk.*;
import org.openscience.cdk.templates.*;
import org.openscience.cdk.tools.*;
import org.openscience.cdk.tools.manipulator.*;
import org.openscience.cdk.qsar.descriptors.molecular.*;
import org.openscience.cdk.qsar.result.*;

adder = CDKHydrogenAdder.getInstance(
  DefaultChemObjectBuilder.getInstance()
);
oxazone = MoleculeFactory.makeOxazole();
benzene = MoleculeFactory.makeBenzene();
// add explicit hydrogens ...
AtomContainerManipulator.percieveAtomTypesAndConfigureAtoms(oxazone)
adder.addImplicitHydrogens(oxazone)
AtomContainerManipulator.convertImplicitToExplicitHydrogens(oxazone)
AtomContainerManipulator.percieveAtomTypesAndConfigureAtoms(benzene)
adder.addImplicitHydrogens(benzene)
AtomContainerManipulator.convertImplicitToExplicitHydrogens(benzene)
descriptor = new TPSADescriptor()
println "TPSA of oxazone: " +
  ((DoubleResult)descriptor.calculate(oxazone).value)
  .doubleValue()
println "TPSA of benzene: " +
  ((DoubleResult)descriptor.calculate(benzene).value)
  .doubleValue()
```
**Output:**
```plain
TPSA of oxazone: 21.59
TPSA of benzene: 0.0
```
