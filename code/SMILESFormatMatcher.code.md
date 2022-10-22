# SMILESFormatMatcher.java
**Source code:**
```java
import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.io.formats.*;
import org.openscience.cdk.io.formats.IChemFormatMatcher.MatchResult;
import org.openscience.cdk.io.*;
import org.openscience.cdk.silent.*;
import org.openscience.cdk.smiles.*;
import org.openscience.cdk.exception.InvalidSmilesException;
import java.util.List;

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
 public static void main(String[] args) {}
}
```
**Output:**
```plain
```
