<section label="ch:depiction" level="#">Depiction</section>

The CDK originates from the merger of Jmol and <topic>JChemPaint</topic> [<cite>Q27093381</cite>]. As such, CDK has long
contained code to depict molecules. However, after the 1.0 series, a rewrite of the code base
was initiated, causing the CDK 1.2 series to not be available with <topic>rendering</topic> functionality.
During the development of the 1.4 series, the rendering code became gradually available as
a set of patches, and, separately, as a JChemPaint applet. The new rendering code has
entered the CDK.

However, if you need rendering of reaction schemes or the editing functionality found
in JChemPaint, you still need the <topic>CDK-JChemPaint</topic> patch.

## Molecules

Rendering molecules to an image was previously done in a few steps. It involved 
a <class>BasicSceneGenerator</class>, a <class>BasicAtomGenerator</class>, and a
<class>BasicBondGenerator</class>.
Nowadays, we can use the convenience class `DepictionGenerator`.
The code example looks like this:

<code>RenderMolecule</code>

This results in the image of triazole given in Figure <xref>fig:triazole</xref>.

<figure label="fig:triazole" caption="2D diagram of triazole">
![](images/generated/RenderMolecule.png) <br />
</figure>

## Background color

Starting from the above pattern, you can also customize the <topic>background color</topic>.
This code uses the `withParam()` method to customize the rendering:

<code>BackgroundColor</code>

The result of this code is depicted in Figure <xref>fig:backgroundColor</xref>.

<figure label="fig:backgroundColor" caption="Triazole depicted with a custom, grey background.">
![](images/generated/BackgroundColor.png) <br />
</figure>

## References

<references/>
