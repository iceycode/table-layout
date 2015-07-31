**Note: This documentation is for TableLayout v1, which has been deprecated.**

**See [this blog post](http://www.badlogicgames.com/wordpress/?p=2483) for more information.**

TableLayout is a lightweight library for describing a hierarchy of GUI widgets and laying them out using rows and columns. Assembling GUI widgets in code is clumsy and obfuscates the widget hierarchy. TableLayout provides its own simple and concise language for expressing the GUI. TableLayout cleanly describes an entire object graph of GUI widgets, leaving the Java code clean and uncluttered.

The core of TableLayout is GUI toolkit agnostic and implementations are available for Swing, Android, [libgdx](http://code.google.com/p/libgdx/), and [TWL](http://twl.l33tlabs.org/). This documentation occasionally refers to Swing for examples, but applies to all implementations.
## Editor ##

This Swing application allows a layout to be edited and the resulting GUI is displayed as you type:

[TableLayout Editor](http://table-layout.googlecode.com/svn/wiki/jws/editor.jnlp)

It supports syntax highlighting, on the fly compilation, error underlining, and code completion (ctrl+space). While the editor is Swing, it can be used to prototype and debug layouts for any supported GUI toolkit. Any of the examples on this page can be pasted into the editor to see how they are actually rendered.

A few naming conventions are used to make widget references appear as the appropriate widget. For example, "`[editButton]`" will appear as a JButton because it ends with "Button". Otherwise, widget references will simply appear as labels.

## Quickstart ##

Here is a quick example of a simple form in Swing:

```
JTextField nameEdit = new JTextField();
JTextField addressEdit = new JTextField();
...
Table table = new Table();
TableLayout layout = table.layout;
layout.register("nameEdit", nameEdit);
layout.register("addressEdit", addressEdit);
layout.parse("'Name:' [nameEdit] width:100 --- 'Address:' [addressEdit] width:100");
...
jframe.setContentPane(table);
```

A `Table` is created, which extends the basic container for the toolkit being used (eg, Component for Swing). The table has a `TableLayout`, which is used to name some widgets. Then `parse` is called on the layout to describe how the widgets should be arranged. A Java API can be used in conjunction with or instead of the string description.

## Language ##

The TableLayout language allows an entire GUI to be described with minimal markup. The language is structured in this way:

```
{
   table-properties
   * default-cell-properties
   | column1-cell-properties | column2-cell-properties ...
   --- row1-properties
   widget cell-properties (widget-section)
   widget cell-properties (widget-section)
   ...
   --- row2-properties
   widget cell-properties (widget-section)
   widget cell-properties (widget-section)
   ...
}
```

Everything in the above is optional, including all whitespace. The format may seem cryptic at first, but actually has a simple, logical structure that is designed to not get in the way.

The first section in the table is the [table properties](#Table_properties.md). These control everything that isn't a cell property, such as padding around the table. Properties consist of a name and zero or more values. Properties are separated by whitespace and are structured in this way:

```
name name:value name:'value' name:value,'value',...
```

Names are case insensitive. Unquoted values can consist of any alphanumeric character, period (`.`), underscore (`_`), or percent (`%`).

After the table properties, an asterisk (`*`) denotes the default [cell properties](#Cell_properties.md) section. All cells in the table will inherit these properties.

The next section allows cell properties to be defined for each column. Properties after the first pipe character (`|`) are for the first column, after the second pipe are for the second column, etc. Column properties will overwrite any default cell properties.

Three minus characters (`---`) denote the start of a new row. This is optional for the first row, if no row properties need to be defined. Row properties will overwrite any default and column properties.

Next comes the definition of the widgets for each cell in the row. The first part, labeled "widget" above, can be specified in multiple ways:

  1. **`[name]`** References a registered widget. If no widget is found, a class with this name and is instantiated if found.
  1. **`[name:class]`** or **`[:class]`** Creates a new instance of the class. If a name is give, the instance is registered.
  1. **`[]`** or **`[name:]`** Creates an empty widget (eg, a JPanel in Swing), typically used to take up space or to later be replaced. If a name is give, the widget is registered.
  1. **`'label'`** or **`[name:'label']`** Creates a new label. If a name is give, the label is registered.
  1. **`{table}`** or **`[name:{table}]`** This defines a nested table. If a name is give, the table is registered.
  1. **`<stack>`** or **`[name:<stack>]`** This defines a [stack widget](#Stacks.md). If a name is give, the stack is registered.

After the widget are the cell properties. These will overwrite any default, column, or row properties.

The last part of a widget definition is the widget section. This is surrounded by parentheses and has this structure:

```
(
   widget-properties
   widget layout-string widget-section
   widget layout-string widget-section
   ...
)
```

The widget properties are name/value pairs (formatted like cell properties) used to call methods or set fields on the widget. If no field or method can be found, the name will be used to find a bean setter method. The property values are automatically converted to the correct types (if possible).

After the widget properties, child widgets can be defined. The format is very similar to adding widgets to a cell, except there are no cell properties since the widget is being added to another widget and not to a table. Instead of cell properties, everything between the widget and widget section is taken literally as a string and used as a parameter when the widget is added (eg, see [Container#add](http://download.oracle.com/javase/6/docs/api/java/awt/Container.html#add%28java.awt.Component,%20java.lang.Object%29)).

## Whitespace ##

All whitespace is optional, except between properties. This allows a GUI to be described in Java code without worrying about missing a space or embedding newlines:

```
layout.register("nameEdit", new JTextField());
layout.register("addressEdit", new JTextField());
table.parse("size:300 align:bottom"
   + "'Name:'"
   + "[nameEdit] expand fill:x"
   + "---"
   + "'Address:'"
   + "[addressEdit] fill:x");
```

## Widget references ##

When widgets are create in Java, the widget must be registered with the Table:

```
JTextField nameEdit = new JTextField();
Table table = new Table();
TableLayout layout = table.layout;
layout.register("nameEdit", nameEdit);
layout.parse("[nameEdit]");
```

When widgets are created with TableLayout markup, the instance of a widget can be gotten by name:

```
Table table = new Table();
TableLayout layout = table.layout;
layout.parse("[nameEdit:JTextField]");
nameEdit = (JTextField)layout.getWidget("nameEdit");
```

The cell for a widget can also be obtained and modified:

```
Cell cell = layout.getCell("nameEdit");
cell.padTop = 10;
layout.invalidate();
```

The `getWidget` and `getCell` methods will find widgets and cells even if they are defined in nested tables.

There are methods available to get all cells or widgets with a given prefix, to replace the widget in a cell, etc.

## Widget size ##

By default, the minimum, maximum, and preferred sizes from the widgets themselves are used. This can be overridden by the width, height, and size cell properties.

If there isn't enough width for all columns to be their preferred size, TableLayout first sizes each column to its minimum width, then allocates the remaining space by using the preferred size ratio. This means that a widget with a (preferred - minimum) size twice as large as another widget will be allocated twice as much of the limited width than the other widget. This ensures widgets are sized optimally when space is at a premium.

## Table size ##

The `Table` instance gets sized by its parent. The widgets are arranged inside of the `Table` container using a table of rows and columns. If this table is smaller than the `Table` container, it is centered by default. This can be changed by the "align" table property.

## Java API ##

Everything that can be described with TableLayout markup can also be done via the Java API. Describing a whole TableLayout via the Java API would be much more verbose than using markup, however sometimes it is convenient to mix the two:

```
Table table = new Table();
TableLayout layout = table.layout;
layout.parse("pad:20 align:top * expand:x fill size:100,50");
for (Button button : buttons) {
    Cell cell = layout.addCell(button);
    ...
}
```

This describes some table properties and default cell properties, then uses the Java API to add widgets.

## Stacks ##

A stack widget is a special kind of container that lays out its children to be the full size of the stack. This is useful when it is necessary to have widgets overlap, such as having some images behind other widgets. The first widget added to the stack is drawn on the bottom, and the last widget added is drawn on the top.

This is the format of a stack:

```
<
   widget (widget-section)
   widget (widget-section)
   ...
>
```

## Child widgets ##

The ability to add widgets to other widgets enables non-table based layouts to be described with TableLayout markup and mixed with table-based layouts. Eg, a tabbed pane with a table in one of the tabs:

```
'Tabbed pane:' align:left
---
[tabs:JTabbedPane] expand fill (
   [tabOne] One
   {
      'Second tab'
      ---
      'has a table!'
   } Two
)
```
![http://table-layout.googlecode.com/svn/wiki/images/11.png](http://table-layout.googlecode.com/svn/wiki/images/11.png)

A split pane with a bean property and a method call:

```
[split:JSplitPane] expand fill (
   dividerSize:25 setResizeWeight:0.4
   'Left widget' left
   {
      'Table on the right!'
      ---
      [someEdit] fill
   } right
)
```
![http://table-layout.googlecode.com/svn/wiki/images/12.png](http://table-layout.googlecode.com/svn/wiki/images/12.png)

If the widget is a Swing LayoutManager, it is automatically wrapped in a JPanel:

```
[:BorderLayout] expand fill (
   'East widget' East
   [centerButton] Center
   {
      'West'
      ---
      'table'
   } West
)
```
![http://table-layout.googlecode.com/svn/wiki/images/13.png](http://table-layout.googlecode.com/svn/wiki/images/13.png)

Note the name was omitted above, since a reference to it was not needed. The colon can also be omitted:

```
[JTabbedPane] expand fill (
   'Tab One' One
   'Tab Two' Two
)
```

The TableLayout would try to find a widget named "JTabbedPane" and fail. It would then look for a class by that name, find it, and create an instance.

Fully qualified class names can be used. Otherwise, TableLayout tries to find the class by using a number if prefixes. For Swing, the default prefixes are "javax.swing." and "java.awt.". More can be added with TableLayout.addClassPrefix(String).

If the name and class are omitted, a blank widget is added (eg, JPanel in Swing).

## Table properties ##

| **Property** | **Synonyms** | **Examples** | **Description** |
|:-------------|:-------------|:-------------|:----------------|
| debug        |              | debug<br>debug:cell<br>debug:table,widget <table><thead><th> Turns on borders for debugging.<br>Values: all,table,cell,widget,none </th></thead><tbody>
<tr><td> size         </td><td>              </td><td> size:10<br>size:10,20 </td><td> Sets the table's width and height. </td></tr>
<tr><td> width        </td><td> w            </td><td> width:10     </td><td> Sets the table's width. </td></tr>
<tr><td> height       </td><td> h            </td><td> height:10    </td><td> Sets the table's height. </td></tr>
<tr><td> align        </td><td>              </td><td> align:top<br>align:top,right<br>align:center,left </td><td> Sets the alignment of the table inside the container.<br>Values: top, left, bottom, right, center </td></tr>
<tr><td> padding      </td><td> pad          </td><td> padding:10<br>padding:10,20,30,40 </td><td> Sets the top, left, bottom, and right padding. </td></tr>
<tr><td> paddingTop   </td><td> padTop<br>padt </td><td> paddingTop:10 </td><td> Sets the top padding. </td></tr>
<tr><td> paddingLeft  </td><td> padLeft<br>padl </td><td> paddingLeft:10 </td><td> Sets the left padding. </td></tr>
<tr><td> paddingBottom </td><td> padBottom<br>padb </td><td> paddingBottom:10 </td><td> Sets the bottom padding. </td></tr>
<tr><td> paddingRight </td><td> padRight<br>padr </td><td> paddingRight:10 </td><td> Sets the right padding. </td></tr></tbody></table>

<h2>Cell properties</h2>

<table><thead><th> <b>Property</b> </th><th> <b>Synonyms</b> </th><th> <b>Examples</b> </th><th> <b>Description</b> </th></thead><tbody>
<tr><td> size            </td><td>                 </td><td> size:10<br>size:10,20 </td><td> Sets the min, preferred, and max width and height. </td></tr>
<tr><td> width           </td><td> w               </td><td> width:10        </td><td> Sets the min, preferred, and max width. </td></tr>
<tr><td> height          </td><td> h               </td><td> height:10       </td><td> Sets the min, preferred, and max height. </td></tr>
<tr><td> minsize         </td><td> min             </td><td> minsize:10<br>minsize:10,20 </td><td> Sets the min width and height. </td></tr>
<tr><td> minwidth        </td><td> minw            </td><td> minwidth:10     </td><td> Sets the min width. </td></tr>
<tr><td> minheight       </td><td> minh            </td><td> minheight:10    </td><td> Sets the min height. </td></tr>
<tr><td> prefsize        </td><td> pref            </td><td> prefsize:10<br>prefsize:10,20 </td><td> Sets the preferred width and height. </td></tr>
<tr><td> prefwidth       </td><td> prefw           </td><td> prefwidth:10    </td><td> Sets the preferred width. </td></tr>
<tr><td> prefheight      </td><td> prefh           </td><td> prefheight:10   </td><td> Sets the preferred height. </td></tr>
<tr><td> maxsize         </td><td> max             </td><td> maxsize:10<br>maxsize:10,20 </td><td> Sets the max width and height. </td></tr>
<tr><td> maxwidth        </td><td> maxw            </td><td> maxwidth:10     </td><td> Sets the max width. </td></tr>
<tr><td> maxheight       </td><td> maxh            </td><td> maxheight:10    </td><td> Sets the max height. </td></tr>
<tr><td> expand          </td><td>                 </td><td> expand<br>expand:x<br>expand:y<br>expand:10<br>expand:10,20 </td><td> Makes the cell take up any remaining width and height.<br>Makes the cell take up any remaining width.<br>Makes the cell take up any remaining height.<br>Sets the weight for taking up any remaining width and height.<br>Sets the weight for taking up any remaining width and height. </td></tr>
<tr><td> fill            </td><td>                 </td><td> fill<br>fill:x<br>fill:y<br>fill:10<br>fill:10,20 </td><td> Sets the widget size to the cell size.<br>Sets the widget width to the cell width.<br>Sets the widget height to the cell height.<br>Sets the widget size to 0-100% of the cell size.<br>Sets the widget size to 0-100% of the cell size. </td></tr>
<tr><td> align           </td><td>                 </td><td> align:top<br>align:top,right<br>align:center,left </td><td> Sets the alignment of the widget inside the cell.<br>Values: top, left, bottom, right, center </td></tr>
<tr><td> colspan         </td><td>                 </td><td> colspan:3       </td><td> Sets the number of columns the cell occupies. </td></tr>
<tr><td> uniform         </td><td>                 </td><td> uniform<br>uniform:x<br>uniform:y<br>uniform:false </td><td> Cells marked uniform will be the same size. </td></tr>
<tr><td> padding         </td><td> pad             </td><td> padding:10<br>padding:10,20<br>padding:10,20,30<br>padding:10,20,30,40 </td><td> Sets the top, left, bottom, and right padding.<br>Sets the top and left padding.<br>Sets the top, left, and bottom padding.<br>Sets the top, left, bottom, and right padding. </td></tr>
<tr><td> paddingTop      </td><td> padTop<br>padt  </td><td> paddingTop:10   </td><td> Sets the top padding. </td></tr>
<tr><td> paddingLeft     </td><td> padLeft<br>padl </td><td> paddingLeft:10  </td><td> Sets the left padding. </td></tr>
<tr><td> paddingBottom   </td><td> padBottom<br>padb </td><td> paddingBottom:10 </td><td> Sets the bottom padding. </td></tr>
<tr><td> paddingRight    </td><td> padRight<br>padr </td><td> paddingRight:10 </td><td> Sets the right padding. </td></tr>
<tr><td> spacing         </td><td> space           </td><td> spacing:10<br>spacing:10,20<br>spacing:10,20,30<br>spacing:10,20,30,40 </td><td> Sets the top, left, bottom, and right spacing.<br>Sets the top and left spacing.<br>Sets the top, left, and bottom spacing.<br>Sets the top, left, bottom, and right spacing. </td></tr>
<tr><td> spacingTop      </td><td> spaceTop<br>spacet </td><td> spacingTop:10   </td><td> Sets the top spacing. </td></tr>
<tr><td> spacingLeft     </td><td> spaceLeft<br>spacel </td><td> spacingLeft:10  </td><td> Sets the left spacing. </td></tr>
<tr><td> spacingBottom   </td><td> spaceBottom<br>spaceb </td><td> spacingBottom:10 </td><td> Sets the bottom spacing. </td></tr>
<tr><td> spacingRight    </td><td> spaceRight<br>spacer </td><td> spacingRight:10 </td><td> Sets the right spacing. </td></tr>
<tr><td> ignore          </td><td>                 </td><td> ignore          </td><td> Causes the cell to be ignored for layout purposes. </td></tr></tbody></table>

<h2>Special property values</h2>

If a size, width, height, padding, or spacing property for a table or cell end with a percent character (%), that percentage of the table's width or height will be used.<br>
<br>
If a size, width, or height property of a cell has a value of "min", "pref", or "max", the minimum, preferred, or maximum size of the widget for that cell will be used.<br>
<br>
Additional special property values can be customized by overriding the <code>width(String)</code> and <code>height(String)</code> methods on the <code>TableLayout</code> class.<br>
<br>
<h2>Customization</h2>

The <code>TableLayout</code> class has many methods that can be overridden to customize behavior. An alternate TableLayout implementation can be specified in the Table constructor:<br>
<br>
<pre><code>Table table = new Table(new TableLayout() {<br>
	public Component newWidget (String className) {<br>
		if (className.equals("button")) return new Button();<br>
		return super.newWidget(className);<br>
	}<br>
<br>
	public void setProperty (Component object, String name, List&lt;String&gt; values) {<br>
		if (object instanceof Button){<br>
			Button button = (Button)object;<br>
			if (name.equals("someproperty")) {<br>
				...<br>
                return;<br>
			}<br>
		}				<br>
		super.setProperty(object, name, values);<br>
	}<br>
});<br>
</code></pre>

In this example, overriding <code>newWidget</code> provides a shortcut for creating a <code>Button</code>. This can be used in layouts instead of a class name, eg "<code>[button] --- [name:button]</code>". Overriding <code>setProperty</code> allows property shortcuts to be used in the TableLayout markup. This allows common code to easily be reused while keeping layouts concise.<br>
<br>
<h2>Step by step</h2>

<pre><code>'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/1.png' />

The square brackets refer to a named widget that TableLayout has been told about in code:<br>
<br>
<pre><code>Table table = new Table()<br>
table.layout.register("nameEdit", new JTextField());<br>
table.layout.register("addressEdit", new JTextField());<br>
</code></pre>

It is easier to see what is going on with the "debug" table property:<br>
<br>
<pre><code>debug<br>
'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/2.png' />

The large dotted rectangle shows the size of the TableLayout. Inside, the solid lines show the table cells and the dotted lines show the widget bounds. The edit fields are so small because JTextField returns 0 for its preferred width.<br>
<br>
The "align" table property aligns the table within the TableLayout. The "expand" cell property causes that cell to take up all extra space.<br>
<br>
<pre><code>debug align:top<br>
'Name:'<br>
[nameEdit] expand:x<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/3.png' />

The "fill" cell property makes the widget the same size as the cell.<br>
<br>
<pre><code>debug align:top<br>
'Name:'<br>
[nameEdit] expand:x fill:x<br>
---<br>
'Address:'<br>
[addressEdit] fill:x<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/4.png' />

This shows how to set cell properties for an entire column. This is especially useful when building forms.<br>
<br>
<pre><code>debug align:top<br>
| align:right | fill:x expand:x<br>
'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/5.png' />

This shows how to set cell properties for all cells. The "padding" property adds space around every cell (probably not what is wanted for this particular GUI).<br>
<br>
<pre><code>debug align:top<br>
* padding:5<br>
| align:right | fill:x expand:x<br>
'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/6.png' />

The "spacing" property is similar to padding, but it is not additive between cells and doesn't apply to the edges of the table. This is useful to have proper gaps between widgets, without having to manually specify all of the gaps.<br>
<br>
<pre><code>debug align:top<br>
* spacing:5<br>
| align:right | fill:x expand:x<br>
'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/7.png' />

This shows how to set cell properties for an entire row.<br>
<br>
<pre><code>debug align:top<br>
* spacing:5<br>
| align:right | fill:x expand:x<br>
--- paddingBottom:10<br>
'Name:'<br>
[nameEdit]<br>
---<br>
'Address:'<br>
[addressEdit]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/8.png' />

This shows the "colspan" property.<br>
<br>
<pre><code>debug align:top<br>
* spacing:5<br>
| align:right | fill:x expand:x<br>
'Name:'<br>
[nameEdit] colspan:2<br>
---<br>
'Address:'<br>
[addressEdit]<br>
[browseButton]<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/9.png' />

TableLayout does not support "rowspan", but this shows how to use a nested table to achieve the same affect. A nested table can be used any time a <code>'label'</code> or <code>[namedWidget]</code> could be used.<br>
<br>
<pre><code>debug align:top<br>
{<br>
   * spacing:5<br>
   | align:right | fill:x expand:x<br>
   ---<br>
   'Name:'<br>
   [nameEdit]<br>
   ---<br>
   'Address:'<br>
   [addressEdit]<br>
} expand:x fill:x<br>
'2 rows!' paddingLeft:10<br>
</code></pre>
<img src='http://table-layout.googlecode.com/svn/wiki/images/10.png' />

This covers many of the ways to set properties on cells, though not all available properties have been shown.<br>
<br>
<h2>Architecture</h2>

TableLayout has no runtime dependencies. The library is made up of two core classes:<br>
<br>
The <code>TableLayoutParser</code> class quickly and efficiently parses TableLayout markup using a FSM generated by <a href='http://www.complang.org/ragel/'>Ragel</a>.<br>
<br>
The <code>BaseTableLayout</code> class is the core of the layout algorithm and has a few methods that must be implemented by each GUI toolkit.<br>
<br>
To support a new GUI toolkit, a class called <code>TableLayout</code> is created that extends <code>BaseTableLayout</code>. Also, a class called <code>Table</code> is created which extends the basic container widget for the GUI toolkit. A <code>Table</code> has an instance of a <code>TableLayout</code>, which it uses to invoke a layout at the appropriate times.<br>
<br>
<h2>Similar libraries</h2>

A few Java, table-based layout managers:<br>
<br>
GridBagLayout can handle complex table-based layouts, but does so via a clunky API.<br>
<br>
<a href='http://java.sun.com/products/jfc/tsc/articles/tablelayout/'>TableLayout (the other one)</a> uses 2D arrays of percentages, sizes, and flags to describe the table and how it should be sized. This approach has the same problems as GridBagLayout.<br>
<br>
<a href='http://pagelayout.sourceforge.net/'>PageLayout</a> uses a more concise Java API to describe the table.<br>
<br>
PnutsLayout (webpage no longer available) was written by Toyokazu Tomatsu as part of <a href='http://en.wikipedia.org/wiki/Pnuts'>Pnuts</a>. TableLayout was originally inspired by PnutsLayout.<br>
<br>
<a href='http://chrriis.free.fr/projects/uihierarchy/index.html'>UIHierarchy</a> was also inspired by PnutsLayout. It is interesting because it is not actually a layout manager, instead it uses a combination of method chaining and constraint strings to more cleanly create UI hierarchies and configure layout parameters.<br>
<br>
<a href='http://www.datadosen.se/riverlayout/'>RiverLayout</a> also uses tags in constraint strings. This is better than GridBagLayout, but still doesn't make the hierarchy of widgets clear.<br>
<br>
<a href='http://www.jgoodies.com/freeware/forms/index.html'>FormLayout</a> is similar to RiverLayout, but more sophisticated.<br>
<br>
<a href='http://www.miglayout.com/'>MIGLayout</a> is even more sophisticated than FormLayout. It attempts to support many kinds of layouts beyond tables and has a somewhat bloated number of features. It has a complex constraint language. It can layout using a grid, border, absolute, etc. Like the other layout managers, MIGLayout still relies on the Java API to describe the object graph.<br>
<br>
<a href='http://java.net/projects/designgridlayout'>DesignGridLayout</a> uses canonical grids. For the most part, widgets are simply added and the ideal table is determined automatically. This cuts down the needed Java code to a minimum and enforces UI guidelines. The downside is that DesignGridLayout does not handle arbitrary table layouts. If a UI problem can be handled using a canonical grid, DesignGridLayout is the most elegant solution. If you want to deviate from a canonical grid, you have no recourse.<br>
<br>
Please feel free to submit additional libraries to be included in this section or suggest better descriptions.