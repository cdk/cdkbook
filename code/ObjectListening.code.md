# ObjectListening.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.11')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;

IChemObjectBuilder builder =
  DefaultChemObjectBuilder.getInstance();
molecule = builder.newInstance(IAtomContainer.class);
listener = new IChemObjectListener() {
  public void stateChanged(IChemObjectChangeEvent event) {
    System.out.println("Event: " + event)
  }
}
molecule.addListener(listener)
IAtom atom1 = builder.newInstance(IAtom.class, "C")
molecule.addAtom(atom1)
atom1.setSymbol("N")
```
**Output:**
```plain
Event: org.openscience.cdk.event.ChemObjectChangeEvent[source=AtomContainer(13...
  63574841, #A:1, AtomRef{Atom(493177695, S:C, AtomType(493177695, FC:0, Isoto...
  pe(493177695, Element(493177695, S:C, AN:6))))})]
Event: org.openscience.cdk.event.ChemObjectChangeEvent[source=Atom(493177695, ...
  S:N, AtomType(493177695, FC:0, Isotope(493177695, Element(493177695, S:N, AN...
  :7))))]
```
