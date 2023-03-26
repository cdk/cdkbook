# GroovyClosureForAllAtoms.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.8')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.silent.SilentChemObjectBuilder;
import org.openscience.cdk.config.Isotopes;

IChemObjectBuilder builder = SilentChemObjectBuilder.getInstance();
IAtomContainer molecule = builder.newInstance(IAtomContainer.class);
molecule.addAtom(builder.newInstance(IAtom.class, "C"));
molecule.getAtom(0).setCharge(-0.4);
molecule.addAtom(builder.newInstance(IAtom.class, "H"));
molecule.getAtom(1).setCharge(0.1);
molecule.addAtom(builder.newInstance(IAtom.class, "H"));
molecule.getAtom(2).setCharge(0.1);
molecule.addAtom(builder.newInstance(IAtom.class, "H"));
molecule.getAtom(3).setCharge(0.1);
molecule.addAtom(builder.newInstance(IAtom.class, "H"));
molecule.getAtom(4).setCharge(0.1);
Isotopes isotopeInfo = Isotopes.getInstance();
def forAllAtoms(molecule, block) {
  for (atom in molecule.atoms()) {
    block(atom)
  }
}
totalCharge = 0.0
forAllAtoms(molecule, { totalCharge += it.getCharge() } )
totalCharge = String.format('%.2f', totalCharge)
println "Total charge: ${totalCharge}"
molWeight = 0.0
forAllAtoms(molecule, {
  molWeight += isotopeInfo.getNaturalMass(it)
} )
molWeight = String.format('%.2f', molWeight)
println "Molecular weight: ${molWeight}"
```
**Output:**
```plain
Total charge: -0.00
Molecular weight: 16.04
```
