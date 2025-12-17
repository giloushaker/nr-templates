If you need help creating a template or want one added to the default list feel free to ask in [Discord](https://discord.gg/cCtqGbugwb)  

# How it works
The XML processing involves two main steps: expanding nodes and replacing placeholders.

1. Expanding Nodes  
XML nodes such as `<forces>...</forces>`, `<units>...</units>`, and `<profiles>...</profiles>` are expanded by replacing them with their child elements. This process is repeated for each of the matching objects, effectively generating multiple sets of the child elements.
For example: This XML with a roster that contains 2 units named Orc:
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
Text within XML nodes or attributes surrounded by {{...}} will be replaced with the corresponding value.
Results in:
```html
<div>Unit</div>
<div>Orc</div>
<div>Unit</div>
<div>Orc</div>
```

# Common
All elements within a list have these fields/values:  
`last`: Indicates if this is the last element in a list  
`length`: The length of the containing list  
`index`: The (zero-based) position of the element in its containing list  

## List Elements

All "list elements" support the `max` attribute. (`max="1"` is useful for headers)  

### `<forces>`
Values `name`, `options`, `hidden`, `primaryCatalogue`    
Lists: `units`, `entries`, `costs`  
### `<units>`
Values: `name`, `customName`, `primary`, `options`, `secondaries`, `type`, `hidden`, `amount`, `primaryCatalogue`, `note`  
Lists: `entries`, `costs`, `profiles`, `rules`, `secondaries`  
Attributes:  
`exclude-primary-categories`: exclude units with matching primary category (case insensitive, separate by `,`)  
`dedupe`: set to "true" to dedupe similar units  
### `<entries>`
Values: `name`, `primary`, `options`, `secondaries`, `type`, `hidden`, `amount`, `primaryCatalogue`    
Lists: `entries`, `costs`, `profiles`, `rules`, `secondaries`
### `<rules>`
Values: `name`, `description`, `hidden`, `page`  
Attributes:  
`exclude-entry-type`: exclude profiles from entries with matching type (case insensitive, separate by `,`)  
`exclude-entries-with-profile`: exclude profiles from entries containing a matching profile type (case insensitive, separate by `,`)  
`recursive`: if `false`, only returns the rules directly from the current entry  
### `<profiles>`
Values:  `name`, `hidden`, `typeName`, `typeId`, `page`, `group`,  
Lists: `characteristics`, `attributes`  
Attributes:  
`include`: include matching typeNames (case insensitive, separate by `,`)  
`exclude`: exclude matching typeNames (case insensitive, separate by `,`)  
`include-rules`: if `true` rules will be converted to the same format as profiles and included
`exclude-entry-type`: exclude profiles from entries with matching type (case insensitive, separate by `,`)  
`exclude-entries-with-profile`:  exclude profiles from entries containing a matching profile type (case insensitive, separate by `,`)  
`grouped`: if `true` creates a group for each profile type, use `<items>` to render each profile within the group  
`recursive`: if `false`, only returns the profiles directly from the current entry  
### `<characteristics>`, `<attributes>`
Values: `name`, `value`, `originalValue`, `typeId`  
Attributes:  
`include`: include matching typeNames (case insensitive, separate by `,`)  
`exclude`: exclude matching typeNames (case insensitive, separate by `,`)  
### `<costs>`
Values: `name`, `value`, `typeId`  
### `<secondaries>`
Values: `name`


### `<if>`, `<else-if>`, `<else>`
This node's childs will only be rendered if it matches a condition, `<or>` and `<and>` may be nested within an `<if>` for more complex logic.  
Types:  
`equals`: `field` is equal to `value`  
`not-equals`: `field` is not equal to `value`  
`match`: `field` matches the regex in `value`; flags can be added in `flags`  
`not-match`:  `field` does not match the regex in `value`; flags can be added in `flags`  
`has-profile`:  `field` (or the current element) has a profile with typeName in `value` (case insensitive, separate by `,`), supports `recursive` attribute set to `true`  
`has-not-profile`:  `field` (or the current element) does not have a profile with typeName in `value` (case insensitive, separate by `,`), supports `recursive` attribute set to `true`  
`not`: `field` is falsy  
(no type): `field` is truthy  

If the field is `length` or `amount`, these types are also supported: `greater`, `atleast`, `less`, `atmost`, `equals`, `not-equals`

### Examples:
Filter out entries without model or stats profiles
```xml
<entries>
  <if type="has-profile" value="model">
    <or type="has-profile" value="stats">
    ...
  </if>
</entries>
```
Render nothing for characteristics that are empty or only have a -
```xml
<characteristics>
   <if field="value" type="match" value="^[-\s]*$">
     <span></span>
   </if>
   <if field="value" type="not-match" value="^[-\s]*$">
     <span>{{value}}</span>
   </if>
</characteristics>
```

### `<set-attribute>`, `<append-attribute>`
Assigns a value to a parent nodes attribute  
Attributes:  
  `name`: the name of the attribute  
  `value`: the value to set  
  `in`: specify the parent(s) to affect, using a css selector  
  `join`: the join string if appending, defaults to a space

### `<set>`  
Assigns a value to a variable  
Attributes:  
  `name`: the name of the attribute  
  `value`: the value to set  
  `fn`: (optional) a function to call, supports:  
    `toLowerCase`, `toUpperCase`, `trim`, `replace`, `replaceAll`, `match`, `matchAll`, `startswith`, `endswith`, `split`, `includes`,  
    `add`, `subtract`, `multiply`, `divide`, `typeof`  
  `string` or `number` or `regex`: followed by a number for the position to specify a call argument, eg `<set fn="match" regex1=".*"/>` is equivalent to `.match(/.*/)`  
  `index`: the index of the return value to use, usefull for regex match  

### `<groupBy>`  
Group multiple elements that have something in common  
Elements rendered within must have a `<by>` element whose string content will be used to group.  
Groups can then be rendered using `<groups>` and then `<items>` within  

### Examples:  
Group models by their characteristics to only display the stats once if multiple models have the same stats  
```xml
    <groupBy>
        <entries>
            <if type="has-profile" value="{{unitProfiles}}">
                <by>
                    <profiles include="{{unitProfiles}}" recursive="false">
                        <characteristics exclude="{{hideCharacteristics}}">{{name}}{{$text}}</characteristics>
                    </profiles>
                </by>
            </if>
        </entries>

        <groups>
            <!-- Model Stats -->
            <items max="1">
                <profiles include="{{unitProfiles}}" recursive="false">
                <characteristics exclude="{{hideCharacteristics}}">
                    <div class="modelCharacteristic">
                        <div>{{name}}</div>
                        <div>{{$text}}</div>
                    </div>
                </characteristics>
                </profiles>
            </items>
            <div class="model" style="display: flex; gap: 1.2rem;">
                <items>
                    <div>
                        <span>{{amount}}x</span>
                        <span class="model-name">{{name}}</span>
                        <div class="model-options">{{options}}</div>
                    </div>
                </items>
            </div>
        </groups>
    </groupBy>
```

### `<dedupe>`  
Dedupes child nodes

### `<sort>`, `<sort-by>`, `<order>`, `<order-by>`
Re-orders child elements using an attribute, if the specified field isnt directly on the child element it will be searched recursively
Attributes:
  `field`: the name of the attribute to use, defaults to "order"  
  `direction`: "ascending" or "descending", defaults to "ascending"
  
### `<script>`  
Supports evaluating JS code, altho with a limited subset of globals to reduce potential abuse.  
To have a variable acessible in the template outside of the script, it needs to be declared as a global within the script, eg:  
```js
const myLocalConstVariable = 1 // Cant be accessed outside of script
let myLocalLetVariable = 1 // Cant be accessed outside of script
myGlobalVariable = 1 // Can be accessed
```
Global variables including arrays can then be used in the template:
```html
<div>
  <script>
    myArray = [{text: "1"}, {text: "2"}]
    myVariable = "Hello"
  </script>
  {{myVariable}}
  <myArray>
    <span>{{text}}</span>
  </myArray>
</div>
```
Results in
```html
<div>
  Hello
  <span>1</span>
  <span>2</span>
</div>
```
You can use `debugger;` or `console.log` to help debug  
You can view the accessible variables with `console.log(scope)`, you may need to open the `[[Prototype]]` to see everything  

Example script to round characteristics to 1 decimal place:  
```html
<characteristics>
  <span class="stat">{{name}}
    <script>
      isNumber = isFinite(Number(characteristic.value))
      roundedValue = isNumber ? Number(characteristic.value).toFixed(1) : characteristic.value;
    </script>
    <strong type="markdown">{{roundedValue}}</strong>
  </span>
</characteristics>
```

# Special Attributes
### Markdown
`type="markdown"`: renders the content as markdown, intended for characteristic & rule content, may cause issues if node has non-text child nodes  
  
#### Example usage:
```xml
<profiles>
 <characteristics>
  <span type="markdown">{{$text}}</span>
 </characteristics>
</profiles>
```

### Card
`type="card"`: adds a 'dont print this card' checkbox and 'move up' / 'move down' buttons

# Special Classes
`class="print-display-none"`: hide the corresponding element when printing  
`class="removable"`: allows the user to hide this element when printing by clicking on it

# Todo:
- [x] `<and>`, `<or>`
- [x] `<else>`
- [x] `<else-if>`
- [x] attribute that generates a "dont print this card" checkbox (would also allow users to print one per page)
- [x] attribute that generates a "move up" and "move down" button
- [x] scripting
- [ ] user defined variables
- [x] markdown support for profiles
- [ ]  abiity to reuse logic


 

