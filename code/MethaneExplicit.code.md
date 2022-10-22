# MethaneExplicit.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.8')

import java.util.List;
import java.awt.*;
import java.awt.image.*;
import javax.imageio.*;
import javax.vecmath.*;
import org.openscience.cdk.*;
import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.layout.*;
import org.openscience.cdk.renderer.*;
import org.openscience.cdk.renderer.font.*;
import org.openscience.cdk.renderer.generators.*;
import org.openscience.cdk.renderer.visitor.*;
import org.openscience.cdk.templates.*;
import org.openscience.cdk.renderer.generators.BasicSceneGenerator.Margin;




int WIDTH = 220;
int HEIGHT = 220;
  IChemObjectBuilder builder = DefaultChemObjectBuilder.getInstance();
  IAtomContainer mol = builder.newInstance(IAtomContainer.class);
  IAtom a1 = builder.newInstance(IAtom.class,"C");
  a1.setPoint2d(new Point2d(4.6783, 0.1497));
  a1.setImplicitHydrogenCount(4)
  mol.addAtom(a1);
  for (i in 1..4) {
    atom = builder.newInstance(IAtom.class, "H")
    mol.addAtom(atom);
    bond = builder.newInstance(IBond.class, a1, atom, IBond.Order.SINGLE)
    mol.addBond(bond);
  }
// the draw area and the image should be the same size
Rectangle drawArea = new Rectangle(WIDTH, HEIGHT);
Image image = new BufferedImage(
  WIDTH, HEIGHT, BufferedImage.TYPE_INT_RGB
);
StructureDiagramGenerator sdg = new StructureDiagramGenerator();
sdg.setMolecule(mol);
sdg.generateCoordinates();
mol = sdg.getMolecule();
// generators make the image elements
List<IGenerator> generators = new ArrayList<IGenerator>();
generators.add(new BasicSceneGenerator());
generators.add(new BasicBondGenerator());
generators.add(new BasicAtomGenerator());
// the renderer needs to have a toolkit-specific font manager
AtomContainerRenderer renderer =
  new AtomContainerRenderer(generators, new AWTFontManager());
// the call to 'setup' only needs to be done on the first paint
renderer.setup(mol, drawArea);
model = renderer.getRenderer2DModel();
model.set(Margin.class, (double)0.1);

// paint the background
Graphics2D g2 = (Graphics2D)image.getGraphics();
g2.setColor(Color.WHITE);
g2.fillRect(0, 0, WIDTH, HEIGHT);
// the paint method also needs a toolkit-specific renderer
renderer.paint(mol, new AWTDrawVisitor(g2));
ImageIO.write(
  (RenderedImage)image, "PNG",
  new File("MethaneExplicit.png")
);
```
**Output:**
```plain
```
