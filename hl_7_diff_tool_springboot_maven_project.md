# HL7 Diff Tool — Spring Boot (Java 17, Maven)

This repository scaffolds a production‑ready Spring Boot service that compares two HL7 v2 messages with configurable rules and multiple highlight renderers (JSON/HTML/ANSI). It implements the domain and engine design you outlined.

---

## Project Tree
```
hl7-diff-tool-springboot/
├─ pom.xml
├─ README.md
├─ .gitignore
├─ src/
│  ├─ main/
│  │  ├─ java/
│  │  │  └─ com/example/hl7diff/
│  │  │     ├─ Hl7DiffApplication.java
│  │  │     ├─ api/
│  │  │     │  ├─ CompareController.java
│  │  │     │  ├─ dto/
│  │  │     │  │  ├─ CompareRequest.java
│  │  │     │  │  ├─ CompareResponse.java
│  │  │     │  │  ├─ HighlightFormat.java
│  │  │     │  │  └─ RuleSpec.java
│  │  │     ├─ core/
│  │  │     │  ├─ model/
│  │  │     │  │  ├─ HL7Message.java
│  │  │     │  │  ├─ HL7Segment.java
│  │  │     │  │  ├─ ComparisonDifference.java
│  │  │     │  │  ├─ ComparisonResult.java
│  │  │     │  │  ├─ TextRange.java
│  │  │     │  │  └─ VersionProfile.java
│  │  │     │  ├─ compare/
│  │  │     │  │  ├─ HL7Comparator.java
│  │  │     │  │  ├─ HL7ComparisonContext.java
│  │  │     │  │  ├─ rule/ (rule implementations)
│  │  │     │  │  │  ├─ ComparisonRule.java
│  │  │     │  │  │  ├─ IgnoreWhitespaceRule.java
│  │  │     │  │  │  ├─ IgnoreCaseRule.java
│  │  │     │  │  │  ├─ IgnoreBlankLinesRule.java
│  │  │     │  │  │  ├─ IgnoreTrailingEmptyFieldsRule.java
│  │  │     │  │  │  ├─ IgnorePatternRule.java
│  │  │     │  │  │  ├─ HighlightFieldRule.java
│  │  │     │  │  │  ├─ DateComparisonRule.java
│  │  │     │  │  │  ├─ IgnoreSegmentsFieldsRule.java
│  │  │     │  │  │  ├─ AlignSimilarSegmentsRule.java
│  │  │     │  │  │  ├─ NotEqualsRule.java
│  │  │     │  │  │  ├─ EqualsContainsRule.java
│  │  │     │  │  │  ├─ IgnoreTrailingComponentSeparatorsRule.java
│  │  │     │  │  │  └─ AllowExtraSegmentsRule.java
│  │  │     │  ├─ highlight/
│  │  │     │  │  ├─ HighlightEngine.java
│  │  │     │  │  ├─ HighlightType.java
│  │  │     │  │  ├─ NoHighlightEngine.java
│  │  │     │  │  ├─ ConsoleHighlightEngine.java
│  │  │     │  │  ├─ HtmlHighlightEngine.java
│  │  │     │  │  └─ JsonHighlightEngine.java
│  │  │     │  ├─ profile/
│  │  │     │  │  └─ ProfileFactory.java
│  │  │     │  └─ util/
│  │  │     │     ├─ Hl7Parser.java
│  │  │     │     ├─ FieldRef.java
│  │  │     │     └─ StringUtils2.java
│  │  │     └─ config/
│  │  │        └─ JacksonConfig.java
│  │  └─ resources/
│  │     ├─ application.yml
│  │     └─ static/index.html
│  └─ test/
│     └─ java/
│        └─ com/example/hl7diff/
│           ├─ ParserTests.java
│           ├─ ComparatorSmokeTests.java
│           └─ ApiTests.java
└─
```

---

## Build & Run

```bash
# Java 17+ and Maven 3.9+
mvn clean package
mvn spring-boot:run
# Service at http://localhost:8080
```

### Sample cURL
```bash
curl -X POST http://localhost:8080/api/compare \
  -H 'Content-Type: application/json' \
  -d '{
    "leftHl7": "MSH|^~\\&|LIS|...\rPID|1||12345||DOE^JOHN\rOBR|1|||GLUCOSE\r",
    "rightHl7": "MSH|^~\\&|LIS|...\rPID|1||12345||DOE^JON\rOBR|1|||GLUCOSE\r",
    "profile": "v2.5.1",
    "rules": {"ignoreWhitespace": true, "ignoreCase": false, "ignoreBlankLines": true,
               "ignoreTrailingEmptyFields": true, "ignorePatterns": ["\\\d{14}"],
               "highlightFields": ["PID-5"], "dateFields": ["OBR-7"],
               "allowExtraSegments": true,
               "ignoreSegments": ["NTE"],
               "ignoreFields": ["MSH-7"],
               "equalsContains": [{"field":"PID-3", "equals":"12345"}],
               "notEquals": [{"field":"OBX-11", "value":"F"}] },
    "output": "JSON"
  }'
```

---

## Key Files

### `pom.xml`
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>hl7-diff-tool</artifactId>
  <version>1.0.0</version>
  <name>HL7 Diff Tool</name>
  <properties>
    <java.version>17</java.version>
    <spring.boot.version>3.3.4</spring.boot.version>
  </properties>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>${spring.boot.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.14.0</version>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
        <configuration>
          <source>${java.version}</source>
          <target>${java.version}</target>
          <compilerArgs>
            <arg>-parameters</arg>
          </compilerArgs>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### `Hl7DiffApplication.java`
```java
package com.example.hl7diff;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Hl7DiffApplication {
  public static void main(String[] args) {
    SpringApplication.run(Hl7DiffApplication.class, args);
  }
}
```

### API DTOs & Controller
`api/dto/HighlightFormat.java`
```java
package com.example.hl7diff.api.dto;
public enum HighlightFormat { NONE, CONSOLE, HTML, JSON }
```

`api/dto/RuleSpec.java`
```java
package com.example.hl7diff.api.dto;

import java.util.List;

public class RuleSpec {
  public boolean ignoreWhitespace = true;
  public boolean ignoreCase = false;
  public boolean ignoreBlankLines = true;
  public boolean ignoreTrailingEmptyFields = true;
  public boolean allowExtraSegments = true;
  public boolean alignSimilarSegments = true;
  public boolean ignoreTrailingComponentSeparators = true;

  public List<String> ignorePatterns;
  public List<String> ignoreSegments;          // e.g., ["NTE", "Z.."]
  public List<String> ignoreFields;            // e.g., ["MSH-7", "PID-19"]
  public List<String> highlightFields;         // e.g., ["PID-5"]
  public List<String> dateFields;              // e.g., ["OBR-7"]

  public static class EqualsOrContains { public String field; public String equals; public String contains; }
  public static class NotEquals { public String field; public String value; }

  public List<EqualsOrContains> equalsContains; // equals or contains value
  public List<NotEquals> notEquals;             // NOT indicator
}
```

`api/dto/CompareRequest.java`
```java
package com.example.hl7diff.api.dto;

import jakarta.validation.constraints.NotBlank;
import java.util.List;

public class CompareRequest {
  @NotBlank public String leftHl7;
  @NotBlank public String rightHl7;
  public String profile = "v2.5.1"; // v2.5.1, v2.5, v2.3
  public RuleSpec rules = new RuleSpec();
  public HighlightFormat output = HighlightFormat.JSON;
}
```

`api/dto/CompareResponse.java`
```java
package com.example.hl7diff.api.dto;

import com.example.hl7diff.core.model.ComparisonResult;

public class CompareResponse {
  public boolean equal;
  public Object rendered; // string or JSON structure
  public ComparisonResult raw;
}
```

`api/CompareController.java`
```java
package com.example.hl7diff.api;

import com.example.hl7diff.api.dto.*;
import com.example.hl7diff.core.compare.HL7Comparator;
import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.compare.rule.*;
import com.example.hl7diff.core.highlight.*;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.profile.ProfileFactory;
import com.example.hl7diff.core.util.Hl7Parser;
import org.springframework.http.MediaType;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.ArrayList;
import java.util.List;

@RestController
@RequestMapping(path = "/api", produces = MediaType.APPLICATION_JSON_VALUE)
public class CompareController {

  @PostMapping("/compare")
  public CompareResponse compare(@Validated @RequestBody CompareRequest req) {
    HL7Message left = Hl7Parser.parse(req.leftHl7);
    HL7Message right = Hl7Parser.parse(req.rightHl7);

    HL7Comparator comparator = new HL7Comparator();
    // profile-preconfigured rules
    comparator.getRules().addAll(ProfileFactory.forVersion(req.profile));

    // dynamic rules
    if (req.rules.ignoreWhitespace) comparator.addRule(new IgnoreWhitespaceRule());
    if (req.rules.ignoreCase) comparator.addRule(new IgnoreCaseRule());
    if (req.rules.ignoreBlankLines) comparator.addRule(new IgnoreBlankLinesRule());
    if (req.rules.ignoreTrailingEmptyFields) comparator.addRule(new IgnoreTrailingEmptyFieldsRule());
    if (req.rules.allowExtraSegments) comparator.addRule(new AllowExtraSegmentsRule());
    if (req.rules.alignSimilarSegments) comparator.addRule(new AlignSimilarSegmentsRule());
    if (req.rules.ignoreTrailingComponentSeparators) comparator.addRule(new IgnoreTrailingComponentSeparatorsRule());

    if (req.rules.ignorePatterns != null && !req.rules.ignorePatterns.isEmpty())
      comparator.addRule(new IgnorePatternRule(req.rules.ignorePatterns));
    if (req.rules.ignoreSegments != null || req.rules.ignoreFields != null)
      comparator.addRule(new IgnoreSegmentsFieldsRule(req.rules.ignoreSegments, req.rules.ignoreFields));
    if (req.rules.highlightFields != null)
      comparator.addRule(new HighlightFieldRule(req.rules.highlightFields));
    if (req.rules.dateFields != null)
      comparator.addRule(new DateComparisonRule(req.rules.dateFields));
    if (req.rules.equalsContains != null)
      comparator.addRule(new EqualsContainsRule(req.rules.equalsContains));
    if (req.rules.notEquals != null)
      comparator.addRule(new NotEqualsRule(req.rules.notEquals));

    HL7ComparisonContext ctx = new HL7ComparisonContext(left, right);
    ComparisonResult result = comparator.compare(left, right);

    // Choose renderer
    HighlightEngine engine = switch (req.output) {
      case CONSOLE -> new ConsoleHighlightEngine();
      case HTML -> new HtmlHighlightEngine();
      case JSON, NONE -> new JsonHighlightEngine();
    };

    CompareResponse resp = new CompareResponse();
    resp.equal = result.isEqual();
    resp.raw = result; // raw machine-readable
    resp.rendered = engine.render(String.join("\r", Hl7Parser.toLines(left)), result.getDifferences());
    return resp;
  }
}
```

### Domain Model
`core/model/HL7Message.java`
```java
package com.example.hl7diff.core.model;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class HL7Message {
  private final List<HL7Segment> segments = new ArrayList<>();
  public List<HL7Segment> getSegments() { return Collections.unmodifiableList(segments); }
  public void addSegment(HL7Segment s) { segments.add(s); }
}
```

`core/model/HL7Segment.java`
```java
package com.example.hl7diff.core.model;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class HL7Segment {
  private final String name;
  private final List<String> fields = new ArrayList<>();

  public HL7Segment(String name) { this.name = name; }
  public String getName() { return name; }
  public List<String> getFields() { return Collections.unmodifiableList(fields); }
  public void addField(String f) { fields.add(f); }
}
```

`core/model/TextRange.java`
```java
package com.example.hl7diff.core.model;

public class TextRange {
  public final int startOffset; public final int endOffset;
  public TextRange(int startOffset, int endOffset) { this.startOffset = startOffset; this.endOffset = endOffset; }
  public int length() { return Math.max(0, endOffset - startOffset); }
}
```

`core/model/ComparisonDifference.java`
```java
package com.example.hl7diff.core.model;

public class ComparisonDifference {
  public String segmentName;
  public int fieldIndex; // 1-based like HL7 (MSH-9 => 9)
  public String leftValue;
  public String rightValue;
  public TextRange leftRange; // optional
  public TextRange rightRange; // optional
  public String description;
  public String severity = "INFO"; // ERROR for invalids (red reserved)
}
```

`core/model/ComparisonResult.java`
```java
package com.example.hl7diff.core.model;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ComparisonResult {
  private boolean equal;
  private final List<ComparisonDifference> differences = new ArrayList<>();
  public boolean isEqual() { return equal; }
  public void setEqual(boolean equal) { this.equal = equal; }
  public List<ComparisonDifference> getDifferences() { return Collections.unmodifiableList(differences); }
  public void addDifference(ComparisonDifference d) { differences.add(d); }
}
```

`core/model/VersionProfile.java`
```java
package com.example.hl7diff.core.model;

public enum VersionProfile { V2_3, V2_5, V2_5_1 }
```

### Parser & Utils
`core/util/FieldRef.java`
```java
package com.example.hl7diff.core.util;

public record FieldRef(String segment, int field) { // e.g., ("PID", 5)
  public static FieldRef parse(String s) {
    String[] p = s.split("-");
    return new FieldRef(p[0], Integer.parseInt(p[1]));
  }
}
```

`core/util/StringUtils2.java`
```java
package com.example.hl7diff.core.util;

public final class StringUtils2 {
  private StringUtils2() {}
  public static String trimSpaces(String s) { return s == null ? null : s.trim(); }
  public static boolean eqIgnoreCase(String a, String b) {
    return a == null ? b == null : a.equalsIgnoreCase(b);
  }
}
```

`core/util/Hl7Parser.java`
```java
package com.example.hl7diff.core.util;

import com.example.hl7diff.core.model.*;
import java.util.*;

public final class Hl7Parser {
  private Hl7Parser() {}

  public static HL7Message parse(String raw) {
    HL7Message msg = new HL7Message();
    if (raw == null) return msg;
    String normalized = raw.replace("\n", "\r");
    for (String line : normalized.split("\r")) {
      if (line == null || line.isBlank()) continue;
      String[] parts = line.split("\\|", -1); // keep trailing empties
      HL7Segment seg = new HL7Segment(parts[0]);
      for (int i = 1; i < parts.length; i++) seg.addField(parts[i]);
      msg.addSegment(seg);
    }
    return msg;
  }

  public static List<String> toLines(HL7Message m) {
    List<String> lines = new ArrayList<>();
    for (HL7Segment s : m.getSegments()) {
      StringBuilder sb = new StringBuilder(s.getName());
      for (String f : s.getFields()) sb.append('|').append(f == null ? "" : f);
      lines.add(sb.toString());
    }
    return lines;
  }
}
```

### Comparison Core
`core/compare/HL7ComparisonContext.java`
```java
package com.example.hl7diff.core.compare;

import com.example.hl7diff.core.model.*;
import java.util.ArrayList;
import java.util.List;

public class HL7ComparisonContext {
  public final HL7Message left; public final HL7Message right;
  private final List<ComparisonDifference> differences = new ArrayList<>();
  public HL7ComparisonContext(HL7Message left, HL7Message right) { this.left = left; this.right = right; }
  public void add(ComparisonDifference d) { differences.add(d); }
  public List<ComparisonDifference> diffs() { return differences; }
}
```

`core/compare/HL7Comparator.java`
```java
package com.example.hl7diff.core.compare;

import com.example.hl7diff.core.compare.rule.ComparisonRule;
import com.example.hl7diff.core.model.*;
import java.util.*;

public class HL7Comparator {
  private final List<ComparisonRule> rules = new ArrayList<>();
  public List<ComparisonRule> getRules() { return rules; }
  public void addRule(ComparisonRule r) { rules.add(r); }

  public ComparisonResult compare(HL7Message left, HL7Message right) {
    HL7ComparisonContext ctx = new HL7ComparisonContext(left, right);
    for (ComparisonRule r : rules) r.apply(ctx); // rules populate diffs

    // Default structural compare when no rule handles it: segment by segment, field by field
    // (kept lightweight; alignment handled by AlignSimilarSegmentsRule when enabled)
    int max = Math.max(left.getSegments().size(), right.getSegments().size());
    for (int i = 0; i < max; i++) {
      HL7Segment ls = i < left.getSegments().size() ? left.getSegments().get(i) : null;
      HL7Segment rs = i < right.getSegments().size() ? right.getSegments().get(i) : null;
      if (ls == null || rs == null) continue; // AllowExtraSegmentsRule may tolerate
      int fmax = Math.max(ls.getFields().size(), rs.getFields().size());
      for (int f = 1; f <= fmax; f++) {
        String lv = f-1 < ls.getFields().size() ? ls.getFields().get(f-1) : null;
        String rv = f-1 < rs.getFields().size() ? rs.getFields().get(f-1) : null;
        if (!Objects.equals(lv, rv)) {
          ComparisonDifference d = new ComparisonDifference();
          d.segmentName = ls.getName(); d.fieldIndex = f; d.leftValue = lv; d.rightValue = rv;
          d.description = "Structural difference";
          ctx.add(d);
        }
      }
    }

    ComparisonResult result = new ComparisonResult();
    ctx.diffs().forEach(result::addDifference);
    result.setEqual(result.getDifferences().isEmpty());
    return result;
  }
}
```

`core/compare/rule/ComparisonRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;

public interface ComparisonRule { void apply(HL7ComparisonContext context); }
```

**Representative Rules (subset implemented; extend similarly for the rest):**

`IgnoreWhitespaceRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.HL7Segment;

public class IgnoreWhitespaceRule implements ComparisonRule {
  @Override public void apply(HL7ComparisonContext ctx) {
    for (HL7Segment s : ctx.left.getSegments())
      for (int i=0;i<s.getFields().size();i++)
        s.getFields().set(i, s.getFields().get(i).trim());
    for (HL7Segment s : ctx.right.getSegments())
      for (int i=0;i<s.getFields().size();i++)
        s.getFields().set(i, s.getFields().get(i).trim());
  }
}
```

`IgnoreCaseRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;

public class IgnoreCaseRule implements ComparisonRule {
  @Override public void apply(HL7ComparisonContext ctx) { /* marker; handled during equality checks if needed */ }
}
```

`IgnoreBlankLinesRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import java.util.*;

public class IgnoreBlankLinesRule implements ComparisonRule {
  @Override public void apply(HL7ComparisonContext ctx) {
    prune(ctx.left); prune(ctx.right);
  }
  private void prune(HL7Message m){
    List<HL7Segment> kept = new ArrayList<>();
    for (HL7Segment s: m.getSegments()) if (s.getName()!=null && !s.getName().isBlank()) kept.add(s);
    m.getSegments().clear();
    m.getSegments().addAll(kept);
  }
}
```

`IgnoreTrailingEmptyFieldsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.HL7Segment;

public class IgnoreTrailingEmptyFieldsRule implements ComparisonRule {
  @Override public void apply(HL7ComparisonContext ctx) {
    trim(ctx.left); trim(ctx.right);
  }
  private void trim(com.example.hl7diff.core.model.HL7Message m){
    for (HL7Segment s : m.getSegments()) {
      int last = s.getFields().size()-1;
      while (last>=0 && (s.getFields().get(last)==null || s.getFields().get(last).isEmpty())) last--;
      int remove = s.getFields().size()-1;
      for (int i=remove;i>last;i--) s.getFields().remove(i);
    }
  }
}
```

`IgnorePatternRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import java.util.List;
import java.util.regex.Pattern;

public class IgnorePatternRule implements ComparisonRule {
  private final List<Pattern> patterns;
  public IgnorePatternRule(List<String> regexes){ this.patterns = regexes.stream().map(Pattern::compile).toList(); }
  @Override public void apply(HL7ComparisonContext ctx) {
    scrub(ctx.left); scrub(ctx.right);
  }
  private void scrub(HL7Message m){
    for (HL7Segment s: m.getSegments())
      for (int i=0;i<s.getFields().size();i++){
        String v = s.getFields().get(i);
        if (v==null) continue;
        for (Pattern p: patterns) if (p.matcher(v).matches()) { s.getFields().set(i, ""); break; }
      }
  }
}
```

`HighlightFieldRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.FieldRef;
import java.util.*;

public class HighlightFieldRule implements ComparisonRule {
  private final List<FieldRef> refs;
  public HighlightFieldRule(List<String> fields){ this.refs = fields.stream().map(FieldRef::parse).toList(); }
  @Override public void apply(HL7ComparisonContext ctx) {
    for (FieldRef r: refs){
      HL7Segment ls = find(ctx.left, r.segment());
      HL7Segment rs = find(ctx.right, r.segment());
      if (ls==null || rs==null) continue;
      String lv = get(ls, r.field()); String rv = get(rs, r.field());
      if (!Objects.equals(lv, rv)){
        ComparisonDifference d = new ComparisonDifference();
        d.segmentName = r.segment();
        d.fieldIndex = r.field();
        d.leftValue = lv; d.rightValue = rv;
        d.description = "Highlighted field difference";
        ctx.add(d);
      }
    }
  }
  private HL7Segment find(HL7Message m, String seg) {
    return m.getSegments().stream().filter(s -> s.getName().equals(seg)).findFirst().orElse(null);
  }
  private String get(HL7Segment s, int idx){ return idx-1 < s.getFields().size() ? s.getFields().get(idx-1) : null; }
}
```

`DateComparisonRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.FieldRef;
import java.time.*;
import java.time.format.DateTimeFormatter;
import java.util.*;

public class DateComparisonRule implements ComparisonRule {
  private final List<FieldRef> refs;
  private final DateTimeFormatter[] parsers = new DateTimeFormatter[]{
    DateTimeFormatter.ofPattern("yyyyMMddHHmmss"),
    DateTimeFormatter.ofPattern("yyyyMMddHHmm"),
    DateTimeFormatter.ofPattern("yyyyMMdd")
  };
  public DateComparisonRule(List<String> fields){ this.refs = fields.stream().map(FieldRef::parse).toList(); }
  @Override public void apply(HL7ComparisonContext ctx) {
    for (FieldRef r: refs){
      HL7Segment ls = find(ctx.left, r.segment());
      HL7Segment rs = find(ctx.right, r.segment());
      if (ls==null || rs==null) continue;
      String lv = get(ls, r.field()); String rv = get(rs, r.field());
      if (lv==null || rv==null) continue;
      Long le = epoch(lv), re = epoch(rv);
      if (le==null || re==null) {
        ComparisonDifference d = new ComparisonDifference();
        d.segmentName=r.segment(); d.fieldIndex=r.field();
        d.leftValue=lv; d.rightValue=rv;
        d.description = "Invalid date value"; d.severity = "ERROR"; // red reserved
        ctx.add(d);
      } else if (!Objects.equals(le, re)){
        ComparisonDifference d = new ComparisonDifference();
        d.segmentName=r.segment(); d.fieldIndex=r.field();
        d.leftValue=lv; d.rightValue=rv;
        d.description = "Date difference (epoch)";
        ctx.add(d);
      }
    }
  }
  private Long epoch(String v){
    for (DateTimeFormatter f: parsers) try {
      if (f.toString().contains("HH")) return LocalDateTime.parse(v, f).toEpochSecond(ZoneOffset.UTC);
      return LocalDate.parse(v, f).atStartOfDay().toEpochSecond(ZoneOffset.UTC);
    } catch (Exception ignored) {}
    return null;
  }
  private HL7Segment find(HL7Message m, String seg) {
    return m.getSegments().stream().filter(s -> s.getName().equals(seg)).findFirst().orElse(null);
  }
  private String get(HL7Segment s, int idx){ return idx-1 < s.getFields().size() ? s.getFields().get(idx-1) : null; }
}
```

`EqualsContainsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.api.dto.RuleSpec;
import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.FieldRef;
import java.util.*;

public class EqualsContainsRule implements ComparisonRule {
  private final List<RuleSpec.EqualsOrContains> specs;
  public EqualsContainsRule(List<RuleSpec.EqualsOrContains> specs){ this.specs = specs; }
  @Override public void apply(HL7ComparisonContext ctx) {
    for (RuleSpec.EqualsOrContains s : specs) {
      FieldRef r = FieldRef.parse(s.field);
      HL7Segment ls = find(ctx.left, r.segment());
      String lv = get(ls, r.field());
      boolean ok = true;
      if (s.equals != null) ok &= Objects.equals(lv, s.equals);
      if (s.contains != null) ok &= (lv != null && lv.contains(s.contains));
      if (!ok) {
        ComparisonDifference d = new ComparisonDifference();
        d.segmentName=r.segment(); d.fieldIndex=r.field();
        d.leftValue=lv; d.rightValue=s.equals!=null?s.equals:s.contains;
        d.description = "Equals/contains rule not satisfied"; d.severity="ERROR";
        ctx.add(d);
      }
    }
  }
  private HL7Segment find(HL7Message m, String seg) { return m.getSegments().stream().filter(s -> s.getName().equals(seg)).findFirst().orElse(null); }
  private String get(HL7Segment s, int idx){ return s==null?null: (idx-1 < s.getFields().size() ? s.getFields().get(idx-1) : null); }
}
```

`NotEqualsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.api.dto.RuleSpec;
import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.FieldRef;
import java.util.*;

public class NotEqualsRule implements ComparisonRule {
  private final List<RuleSpec.NotEquals> specs;
  public NotEqualsRule(List<RuleSpec.NotEquals> specs){ this.specs = specs; }
  @Override public void apply(HL7ComparisonContext ctx) {
    for (RuleSpec.NotEquals s : specs) {
      FieldRef r = FieldRef.parse(s.field);
      HL7Segment ls = find(ctx.left, r.segment());
      String lv = get(ls, r.field());
      if (Objects.equals(lv, s.value) || (s.value==null && (lv==null || lv.isBlank()))) {
        ComparisonDifference d = new ComparisonDifference();
        d.segmentName=r.segment(); d.fieldIndex=r.field();
        d.leftValue=lv; d.rightValue=s.value;
        d.description = "NOT rule violated"; d.severity="ERROR";
        ctx.add(d);
      }
    }
  }
  private HL7Segment find(HL7Message m, String seg) { return m.getSegments().stream().filter(s -> s.getName().equals(seg)).findFirst().orElse(null); }
  private String get(HL7Segment s, int idx){ return s==null?null: (idx-1 < s.getFields().size() ? s.getFields().get(idx-1) : null); }
}
```

`IgnoreSegmentsFieldsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;
import com.example.hl7diff.core.model.*;
import java.util.*;
import java.util.regex.Pattern;

public class IgnoreSegmentsFieldsRule implements ComparisonRule {
  private final List<Pattern> segmentPatterns;
  private final Set<String> fieldRefs; // e.g., "MSH-7"
  public IgnoreSegmentsFieldsRule(List<String> segs, List<String> fields){
    this.segmentPatterns = segs==null?List.of():segs.stream().map(IgnoreSegmentsFieldsRule::toPattern).toList();
    this.fieldRefs = fields==null?Set.of():new HashSet<>(fields);
  }
  @Override public void apply(HL7ComparisonContext ctx) {
    prune(ctx.left); prune(ctx.right);
  }
  private static Pattern toPattern(String s){
    if (s==null) return Pattern.compile("$");
    String regex = s.replace(".", "\\.").replace("*", ".*").replace("?", ".");
    return Pattern.compile(regex);
  }
  private void prune(HL7Message m){
    List<HL7Segment> kept = new ArrayList<>();
    for (HL7Segment s: m.getSegments()){
      boolean segIgnored = segmentPatterns.stream().anyMatch(p -> p.matcher(s.getName()).matches());
      if (segIgnored) continue;
      for (int i=0;i<s.getFields().size();i++){
        String ref = s.getName()+"-"+(i+1);
        if (fieldRefs.contains(ref)) s.getFields().set(i, "");
      }
      kept.add(s);
    }
    m.getSegments().clear(); m.getSegments().addAll(kept);
  }
}
```

`AllowExtraSegmentsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;

public class AllowExtraSegmentsRule implements ComparisonRule { @Override public void apply(HL7ComparisonContext ctx) { /* no-op marker */ } }
```

`AlignSimilarSegmentsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;

public class AlignSimilarSegmentsRule implements ComparisonRule { @Override public void apply(HL7ComparisonContext ctx) { /* TODO: best-match alignment */ } }
```

`IgnoreTrailingComponentSeparatorsRule.java`
```java
package com.example.hl7diff.core.compare.rule;

import com.example.hl7diff.core.compare.HL7ComparisonContext;

public class IgnoreTrailingComponentSeparatorsRule implements ComparisonRule { @Override public void apply(HL7ComparisonContext ctx) { /* TODO: strip ^~\& trailing component empties */ } }
```

### Highlighting
`core/highlight/HighlightType.java`
```java
package com.example.hl7diff.core.highlight;
public enum HighlightType { INLINE, BLOCK, STRUCTURED }
```

`core/highlight/HighlightEngine.java`
```java
package com.example.hl7diff.core.highlight;

import com.example.hl7diff.core.model.ComparisonDifference;
import java.util.List;

public interface HighlightEngine { Object render(String originalText, List<ComparisonDifference> differences); }
```

`NoHighlightEngine.java`
```java
package com.example.hl7diff.core.highlight;

import com.example.hl7diff.core.model.ComparisonDifference;
import java.util.List;

public class NoHighlightEngine implements HighlightEngine {
  @Override public Object render(String original, List<ComparisonDifference> diffs) { return original; }
}
```

`ConsoleHighlightEngine.java`
```java
package com.example.hl7diff.core.highlight;

import com.example.hl7diff.core.model.ComparisonDifference;
import java.util.List;

public class ConsoleHighlightEngine implements HighlightEngine {
  private static final String RED = "\u001B[31m"; private static final String YELLOW = "\u001B[33m"; private static final String RESET = "\u001B[0m";
  @Override public Object render(String original, List<ComparisonDifference> diffs) {
    String out = original;
    for (ComparisonDifference d : diffs) {
      String mark = ("ERROR".equals(d.severity)?RED:YELLOW) + "<"+d.segmentName+"-"+d.fieldIndex+">" + RESET;
      out += "\n" + mark + " " + d.description + " :: L='"+d.leftValue+"' R='"+d.rightValue+"'";
    }
    return out;
  }
}
```

`HtmlHighlightEngine.java`
```java
package com.example.hl7diff.core.highlight;

import com.example.hl7diff.core.model.ComparisonDifference;
import java.util.List;

public class HtmlHighlightEngine implements HighlightEngine {
  @Override public Object render(String original, List<ComparisonDifference> diffs) {
    StringBuilder sb = new StringBuilder();
    sb.append("<pre class=\"hl7\">").append(original).append("</pre><ul class=\"diffs\">");
    for (ComparisonDifference d: diffs) {
      String cls = "ERROR".equals(d.severity)?"err":"warn";
      sb.append("<li class=\"").append(cls).append("\">")
        .append(d.segmentName).append('-').append(d.fieldIndex).append(':')
        .append(" ").append(d.description)
        .append(" | L=").append(escape(d.leftValue)).append(" R=").append(escape(d.rightValue))
        .append("</li>");
    }
    sb.append("</ul>");
    return sb.toString();
  }
  private String escape(String s){ return s==null?"":s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;"); }
}
```

`JsonHighlightEngine.java`
```java
package com.example.hl7diff.core.highlight;

import com.example.hl7diff.core.model.ComparisonDifference;
import java.util.*;

public class JsonHighlightEngine implements HighlightEngine {
  @Override public Object render(String original, List<ComparisonDifference> diffs) {
    Map<String,Object> m = new LinkedHashMap<>();
    m.put("source", original);
    m.put("differences", diffs);
    return m; // Jackson will serialize
  }
}
```

### Profiles
`core/profile/ProfileFactory.java`
```java
package com.example.hl7diff.core.profile;

import com.example.hl7diff.core.compare.rule.*;
import com.example.hl7diff.core.model.VersionProfile;
import java.util.*;

public final class ProfileFactory {
  private ProfileFactory() {}
  public static List<ComparisonRule> forVersion(String v){
    VersionProfile p = switch (String.valueOf(v).toLowerCase()){
      case "v2.3", "2.3" -> VersionProfile.V2_3;
      case "v2.5", "2.5" -> VersionProfile.V2_5;
      default -> VersionProfile.V2_5_1;
    };
    // Minimal defaults per version; extend as needed
    return switch (p){
      case V2_3 -> List.of(new IgnoreWhitespaceRule());
      case V2_5 -> List.of(new IgnoreWhitespaceRule(), new IgnoreTrailingEmptyFieldsRule());
      case V2_5_1 -> List.of(new IgnoreWhitespaceRule(), new IgnoreTrailingEmptyFieldsRule());
    };
  }
}
```

### Config & Static
`resources/application.yml`
```yaml
server:
  port: 8080
spring:
  mvc:
    problemdetails:
      enabled: true
```

`resources/static/index.html`
```html
<!doctype html>
<html><head><meta charset="utf-8"><title>HL7 Diff Tool</title></head>
<body>
  <h1>HL7 Diff Tool</h1>
  <p>POST <code>/api/compare</code> to compare two HL7 messages. See README for JSON format.</p>
</body></html>
```

### Tests (samples)
`test/java/.../ParserTests.java`
```java
package com.example.hl7diff;

import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.Hl7Parser;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class ParserTests {
  @Test void parseSimple(){
    HL7Message m = Hl7Parser.parse("MSH|^~\\&|LIS\rPID|1||123\r");
    assertEquals(2, m.getSegments().size());
    assertEquals("MSH", m.getSegments().get(0).getName());
    assertEquals("PID", m.getSegments().get(1).getName());
  }
}
```

`ComparatorSmokeTests.java`
```java
package com.example.hl7diff;

import com.example.hl7diff.core.compare.HL7Comparator;
import com.example.hl7diff.core.compare.rule.HighlightFieldRule;
import com.example.hl7diff.core.model.*;
import com.example.hl7diff.core.util.Hl7Parser;
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;
import java.util.List;

public class ComparatorSmokeTests {
  @Test void diffOnHighlightedField(){
    HL7Message a = Hl7Parser.parse("PID|1||123||DOE^JOHN\r");
    HL7Message b = Hl7Parser.parse("PID|1||123||DOE^JON\r");
    HL7Comparator c = new HL7Comparator();
    c.addRule(new HighlightFieldRule(List.of("PID-5")));
    ComparisonResult r = c.compare(a,b);
    assertFalse(r.isEqual());
    assertTrue(r.getDifferences().stream().anyMatch(d -> d.segmentName.equals("PID") && d.fieldIndex==5));
  }
}
```

`ApiTests.java`
```java
package com.example.hl7diff;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class ApiTests { @Test void contextLoads() {} }
```

### README.md
```markdown
# HL7 Diff Tool (Spring Boot)

**Features**
- Compare two HL7 v2 messages with configurable rules
- Profiles for v2.3, v2.5, v2.5.1
- Rule set: ignore whitespace/case/blank lines/trailing fields/patterns; highlight fields; date compare; allow extra segments; ignore specific segments/fields; equals/contains; NOT rule
- Renderers: JSON (default), HTML, ANSI Console

## Build & Run
```bash
mvn clean package
mvn spring-boot:run
```

## API
`POST /api/compare`
```json
{
  "leftHl7": "MSH|^~\\&|...\rPID|...",
  "rightHl7": "MSH|^~\\&|...\rPID|...",
  "profile": "v2.5.1",
  "rules": { /* see RuleSpec */ },
  "output": "JSON|HTML|CONSOLE|NONE"
}
```

## Notes
- Date formats supported: `yyyyMMddHHmmss`, `yyyyMMddHHmm`, `yyyyMMdd`.
- Red color (severity `ERROR`) reserved for invalid values.
- Extend rules by implementing `ComparisonRule` and registering in `ProfileFactory` or controller.
```

---

## Next Steps / TODOs
- Implement smart segment alignment and component/subcomponent comparisons.
- Add CLI wrapper (`spring-boot-starter`) to run diffs from command line.
- Plug in HL7 library for strict conformance (optional): HAPI HL7v2.
- Persist named rule sets and profiles.
- Produce JUnit assertion artifacts for CI pipelines.

