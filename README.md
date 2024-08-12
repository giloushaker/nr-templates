# How it works
The XML processing involves two main steps: expanding nodes and replacing placeholders.

1. Expanding Nodes  
XML nodes such as `<forces>...</forces>`, `<units>...</units>`, and `<profiles>...</profiles>` are expanded by replacing them with their child elements. This process is repeated for each of the matching objects, effectively generating multiple sets of the child elements.
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
Within nodes textContent or Attributes, any text wrapped in {{...}} will be replaced with the corresponding value. For example:
```xml
<profiles>
  <span>{{name}}</span>
</profiles>
```
Results in (for each matching profile):
```html
<span>Example</span>
```

# Common
All elements have these fields/values:  
`first`: Indicates if this is the first element in a list  
`last`: Indicates if this is the last element in a list  
`odd`: Indicates if this is an odd element in a list  
`even`: Indicates if this is an even element in a list  
`length`: The length of the containing list  
`index`: The (zero-based) position of the element in its containing list  

## List Elements

All "list elements" support the `max` attribute. (`max="1"` is useful for headers)  

### `<forces>`
Values `name`, `options`, `hidden`, `primaryCatalogue`    
Lists: `units`, `entries`, `costs`  
### `<units>`
Values: `name`, `primary`, `options`, `secondaries`, `type`, `hidden`, `amount`, `primaryCatalogue`  
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
Lists: `characteristics`  
Attributes:  
`include`: include matching typeNames (case insensitive, separate by `,`)  
`exclude`: exclude matching typeNames (case insensitive, separate by `,`)  
`exclude-entry-type`: exclude profiles from entries with matching type (case insensitive, separate by `,`)  
`exclude-entries-with-profile`:  exclude profiles from entries containing a matching profile type (case insensitive, separate by `,`)  
`grouped`: creates a group for each profile type, use `<items>` to render each profile within the group  
`recursive`: if `false`, only returns the profiles directly from the current entry  
### `<characteristics>`
Values: `name`, `value`, `originalValue`, `typeId`  
Attributes:  
`include`: include matching typeNames (case insensitive, separate by `,`)  
`exclude`: exclude matching typeNames (case insensitive, separate by `,`)  
### `<costs>`
Values: `name`, `value`, `typeId`  
### `<secondaries>`
Values: `name`


### `<if>`
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
`type="card"`: adds a 'dont print this card' checkbox and support for 'one card per page' option

# Todo:
- [x] `<and>`, `<or>`
- [ ] `<else>`
- [ ] `<else-if>`
- [x] attribute that generates a "dont print this card" checkbox (would also allow users to print one per page)
- [ ] attribute that generates a "move up" and "move down" button
- [ ] mongodb aggregation like way to query the roster
- [ ] variables that are defined by the template and can be provided by the user/gameSystem and used in template / css
- [ ] string manipulation (not sure how to do it, maybe allow scripting?)
- [x] markdown support for profiles
- [ ] a way to get the length of a 'list element' from outside of it, current workaround is to use conditional display:none css




