# How it works
The XML processing involves two main steps: expanding nodes and replacing placeholders.

1. Expanding Nodes
XML nodes like `<forces>...</forces>`, `<units>...</units>`, and `<profiles>...</profiles>` are expanded by replacing them with their child elements. This process is repeated for each instance in the current context, effectively generating multiple sets of the child elements.
For example: a roster with 2 units with this XML:
```xml
<units>
  <div>Unit</div>
  <div>{{name}}</div>
</units>
```
Will be expanded to
```xml
<div>Unit</div>
<div>{{name}}</div>
<div>Unit</div>
<div>{{name}}</div>
```

2. Replacing Placeholders
Within XML nodes, any text wrapped in {{...}} will be replaced with the corresponding value. For example:
```xml
<profiles>
  <span>{{name}}</span>
</profiles>
```
Results in (for each matching profile):
```html
<span>Example</span>
```

# Forces
# Units
# Entry
# Characteristics
# Profiles
# Rules
# Costs
# Secondaries
# Filter / If
