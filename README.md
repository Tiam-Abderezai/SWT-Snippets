import org.eclipse.swt.SWT;
import org.eclipse.swt.dnd.*;
import org.eclipse.swt.layout.*;
import org.eclipse.swt.widgets.*;

public class LeftTabsSimplified {

  // tiny model
  static class ItemData {
    String name, field1;
    ItemData(String n) { name = n; field1 = n + " Data"; }
    @Override public String toString() { return name + "\n" + field1; }
    static ItemData from(String s) {
      String[] p = s.split("\n", 2);
      ItemData d = new ItemData(p.length > 0 ? p[0] : "Item");
      d.field1 = p.length > 1 ? p[1] : d.field1;
      return d;
    }
  }

  public static void main(String[] args) {
    Display dsp = new Display();
    Shell shell = new Shell(dsp);
    shell.setText("Left Tabs (Simplified)");
    shell.setLayout(new GridLayout(2, false));

    // left: tree
    Tree tree = new Tree(shell, SWT.BORDER | SWT.SINGLE | SWT.V_SCROLL);
    tree.setLayoutData(new GridData(SWT.FILL, SWT.FILL, true, true));

    // right: editor
    Text editor = new Text(shell, SWT.BORDER | SWT.MULTI | SWT.WRAP | SWT.V_SCROLL);
    editor.setLayoutData(new GridData(SWT.FILL, SWT.FILL, true, true));

    // seed
    for (int i = 1; i <= 3; i++) add(tree, new ItemData("Item" + i));

    // selection -> load
    tree.addListener(SWT.Selection, e -> {
      TreeItem it = (TreeItem)e.item;
      ItemData d = (ItemData) it.getData();
      editor.setData("bound", it);
      editor.setText(d.field1);
    });

    // editor blur -> save
    editor.addListener(SWT.Deactivate, e -> {
      TreeItem it = (TreeItem) editor.getData("bound");
      if (it == null || it.isDisposed()) return;
      ((ItemData) it.getData()).field1 = editor.getText();
    });

    // context menu
    Menu menu = new Menu(tree);
    tree.setMenu(menu);

    MenuItem miNew = new MenuItem(menu, SWT.PUSH);
    miNew.setText("New Item");
    miNew.addListener(SWT.Selection, e -> {
      int idx = 0;
      TreeItem[] sel = tree.getSelection();
      if (sel.length > 0) idx = tree.indexOf(sel[0]) + 1;
      ItemData d = new ItemData("Item" + (tree.getItemCount() + 1));
      TreeItem it = new TreeItem(tree, SWT.NONE, idx);
      it.setText(d.name); it.setData(d);
      tree.setSelection(it);
      editor.setData("bound", it);
      editor.setText(d.field1);
    });

    MenuItem miDel = new MenuItem(menu, SWT.PUSH);
    miDel.setText("Delete Item");
    miDel.addListener(SWT.Selection, e -> {
      TreeItem[] sel = tree.getSelection();
      if (sel.length == 0) return;
      int idx = tree.indexOf(sel[0]);
      sel[0].dispose();
      if (tree.getItemCount() == 0) { editor.setData("bound", null); editor.setText(""); return; }
      int pick = Math.min(idx, tree.getItemCount() - 1);
      TreeItem it = tree.getItem(pick);
      tree.setSelection(it);
      editor.setData("bound", it);
      editor.setText(((ItemData) it.getData()).field1);
    });

    // DnD: move/copy via TextTransfer (serialize as two lines)
    Transfer[] tx = { TextTransfer.getInstance() };

    DragSource drag = new DragSource(tree, DND.DROP_MOVE | DND.DROP_COPY);
    drag.setTransfer(tx);
    drag.addDragListener(new DragSourceAdapter() {
      @Override public void dragSetData(DragSourceEvent e) {
        TreeItem[] s = tree.getSelection();
        if (s.length == 0) return;
        e.data = ((ItemData) s[0].getData()).toString();
      }
      @Override public void dragFinished(DragSourceEvent e) {
        if (e.detail == DND.DROP_MOVE) {
          TreeItem[] s = tree.getSelection();
          if (s.length > 0) s[0].dispose();
        }
      }
    });

    DropTarget drop = new DropTarget(tree, DND.DROP_MOVE | DND.DROP_COPY);
    drop.setTransfer(tx);
    drop.addDropListener(new DropTargetAdapter() {
      @Override public void dragOver(DropTargetEvent e) {
        e.feedback = DND.FEEDBACK_SCROLL | DND.FEEDBACK_SELECT | DND.FEEDBACK_INSERT_AFTER;
      }
      @Override public void drop(DropTargetEvent e) {
        if (!(e.data instanceof String)) return;
        ItemData d = ItemData.from((String)e.data);
        int insert = tree.getItemCount(); // default: end
        if (e.item instanceof TreeItem) insert = tree.indexOf((TreeItem)e.item) + 1;
        TreeItem it = new TreeItem(tree, SWT.NONE, insert);
        it.setText(d.name); it.setData(d);
        tree.setSelection(it);
        editor.setData("bound", it);
        editor.setText(d.field1);
      }
    });

    // select first
    if (tree.getItemCount() > 0) {
      TreeItem it = tree.getItem(0);
      tree.setSelection(it);
      editor.setData("bound", it);
      editor.setText(((ItemData) it.getData()).field1);
    }

    shell.setSize(520, 380);
    shell.open();
    while (!shell.isDisposed()) if (!dsp.readAndDispatch()) dsp.sleep();
    dsp.dispose();
  }

  private static void add(Tree tree, ItemData d) {
    TreeItem it = new TreeItem(tree, SWT.NONE);
    it.setText(d.name);
    it.setData(d);
  }
}










//+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++











package de.esolutions.fw.tools.trace.viewer.ui.log.item.tool;


/*
 * Basic template for app with left hand tabs
 * and custom pane on right
 * by clay.dot.gerrard.at.gmail.dot.com
 */
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;

import org.eclipse.swt.SWT;
import org.eclipse.swt.widgets.Composite;
import org.eclipse.swt.widgets.Display;
import org.eclipse.swt.widgets.Event;
import org.eclipse.swt.widgets.Listener;
import org.eclipse.swt.widgets.Menu;
import org.eclipse.swt.widgets.MenuItem;
import org.eclipse.swt.widgets.Shell;
import org.eclipse.swt.widgets.Text;
import org.eclipse.swt.widgets.Tree;
import org.eclipse.swt.widgets.TreeItem;
import org.eclipse.swt.graphics.Point;
import org.eclipse.swt.graphics.Rectangle;
import org.eclipse.swt.widgets.MessageBox;
import org.eclipse.swt.dnd.*;
import org.eclipse.swt.layout.*;

public class ToolGroupOptionDND {

    // left hand "tab" tree
    static Tree tree;
    // right hand viewer
    //static CompositePane view;
    // temporary TreeItem used to hold data during Drag & Drop
    static TreeItem sourceTreeItem;
    // keep track of number of treeItems created
    static int count = 1;

    public static void main(String[] args) {
        // setup the display & shell & layout
        final Display display = new Display();
        final Shell shell = new Shell(display);
        setupLayout( shell );

        // And here's the tree
        tree = new Tree(shell, SWT.BORDER);
        // filled with some data
        for (int i = 1; i <= 10; i++) {
            TreeItemData myData = new TreeItemData();
            myData.Name = "Item" + i;
            myData.field1 = myData.Name + " Data";
            TreeItem item = new TreeItem(tree, SWT.NONE);

            item.setText(myData.Name);
            item.setData(myData);
        }

        setupDND( display );


        // wrap it up
        shell.open();
        while (!shell.isDisposed()) {
            if (!display.readAndDispatch())
                display.sleep();
        }
        display.dispose();
    }

    private static void setupDND(final Display display)
    {
        // setup DragSource
        DragSource source = new DragSource(tree, DND.DROP_COPY);
        source.setTransfer(new Transfer[] { TreeItemDataTransfer.getInstance() });

        source.addDragListener(new DragSourceAdapter() {



            public void dragStart(DragSourceEvent event) {
                TreeItem[] selection = tree.getSelection();
                if (selection.length > 0 && selection[0].getData() != null) {
                    event.doit = true;
                    sourceTreeItem = selection[0];
                } else {
                    event.doit = false;
                }
            } // end dargStart

            public void dragSetData(DragSourceEvent event) {
                if (TreeItemDataTransfer.getInstance().isSupportedType(event.dataType))
                    event.data = sourceTreeItem.getData();
            } // end dargSetData

            public void dragFinished(DragSourceEvent event) {
                if (event.doit) {
                    sourceTreeItem.dispose();
                }
                sourceTreeItem = null;
            } // end dragFinished
        }); // end DragSource

        // setup DropTarget
        DropTarget target = new DropTarget(tree, DND.DROP_COPY);
        target.setTransfer(new Transfer[] {TreeItemDataTransfer.getInstance() });

        target.addDropListener(new DropTargetAdapter() {
            public void dragEnter(DropTargetEvent event) {
                event.detail = DND.DROP_COPY;
            } // end dragEnter

            public void dragOver(DropTargetEvent event) {
                event.feedback = DND.FEEDBACK_SCROLL;
                // if the drop target is a specific item in the tree
                if (event.item != null) {
                    TreeItem item = (TreeItem) event.item;
                    Point pt = display.map(null, tree, event.x, event.y);
                    Rectangle bounds = item.getBounds();
                    // give visual cue of drop location to user
                    if (pt.y < bounds.y + bounds.height/2) {
                        event.feedback |= DND.FEEDBACK_INSERT_BEFORE;
                    } else {
                        event.feedback |= DND.FEEDBACK_INSERT_AFTER;
                    }
                } else {
                    // set event.item to last item in list & set feedback to after
                }
            } // end dragOver






            public void drop(DropTargetEvent event) {
                try {
                    if (event.data == null) {
                        event.detail = DND.DROP_NONE;
                        return;
                    }

                    TreeItem newItem;
                    // if the dropTarget is a specific item in the tree
                    if (event.item != null) {
                        TreeItem selection = (TreeItem) event.item;
                        Point pt = display.map(null, tree, event.x, event.y);
                        Rectangle bounds = selection.getBounds();
                        //TreeItem[] items = tree.getItems();
                        // find index of selection
                        int index = tree.indexOf(selection);
                        // insert newItem at index
                        if (pt.y < bounds.y + bounds.height/2) {
                            // insert before
                            newItem = new TreeItem(tree, SWT.NONE, index);
                        } else {
                            // insert after
                            newItem = new TreeItem(tree, SWT.NONE, index+1);
                        }
                    } else {
                        // no specific item selected, drop at the end
                        newItem = new TreeItem(tree, SWT.NONE);
                    }

                    TreeItemData myType = (TreeItemData) event.data;
                    newItem.setText(myType.Name);
                    newItem.setData(myType);
                    // set selection otherwise view data gets out of sync
                    tree.setSelection(newItem);
                } catch (RuntimeException e) {
                    e.printStackTrace();
                }
            } // end drop
        }); // end DropTarget
    }

    private static void setupLayout(final Shell shell)
    {
        FillLayout fillLayout = new FillLayout();
        // columns
        fillLayout.type = SWT.HORIZONTAL;
        // break 'em up a little
        fillLayout.spacing = 3;
        fillLayout.marginHeight = 3;
        shell.setLayout(fillLayout);
    }

} // class LeftTabs

class TreeItemData {
    // all the data to be held by each treeItem
    String Name;
    String field1;
} // new members must be explicitly handled in nativeToJava & javaToNative

// Transfer class for handling DND of TreeItemData
class TreeItemDataTransfer extends ByteArrayTransfer {
    private static final String TreeItemData_TRANSFER_NAME = "TreeItemData_TRANSFER";
    private static final int TreeItemData_TRANSFER_ID = registerType (TreeItemData_TRANSFER_NAME);
    private static TreeItemDataTransfer instance = new TreeItemDataTransfer();

    public static TreeItemDataTransfer getInstance() {
        return instance;
    }

    protected String[] getTypeNames() {
        return new String[] { TreeItemData_TRANSFER_NAME };
    }

    protected int[] getTypeIds() {
        return new int[] {TreeItemData_TRANSFER_ID};
    }

    public void javaToNative (Object object, TransferData transferData) {
        if (object == null || !(object instanceof TreeItemData))
            return;

        TreeItemData myType = (TreeItemData) object;

        if (isSupportedType(transferData)) {
            try {
                ByteArrayOutputStream stream = new ByteArrayOutputStream();
                DataOutputStream out = new DataOutputStream(stream);
                // write out each member in your custom dataType
                out.writeUTF(myType.Name);
                out.writeUTF(myType.field1);
                out.close();

                super.javaToNative(stream.toByteArray(), transferData);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    } // end javaToNative

    public Object nativeToJava (TransferData transferData) {
        if (isSupportedType(transferData)) {
            byte[] buffer = (byte[]) super.nativeToJava(transferData);
            if (buffer == null)
                return null;

            TreeItemData myType = new TreeItemData();

            try {
                ByteArrayInputStream stream = new ByteArrayInputStream(buffer);
                DataInputStream in = new DataInputStream(stream);
                // read in each member in your custom dataType
                myType.Name = in.readUTF();
                myType.field1 = in.readUTF();
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
                return null;
            }
            return myType;
        } else {
            return null;
        }
    } // end nativetoJava

} // end TreeItemDataTransfer



//+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++



/*
* Basic template for app with left hand tabs
* and custom pane on right
* by clay.dot.gerrard.at.gmail.dot.com
*/
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;

import org.eclipse.swt.SWT;
import org.eclipse.swt.widgets.Composite;
import org.eclipse.swt.widgets.Display;
import org.eclipse.swt.widgets.Event;
import org.eclipse.swt.widgets.Listener;
import org.eclipse.swt.widgets.Menu;
import org.eclipse.swt.widgets.MenuItem;
import org.eclipse.swt.widgets.Shell;
import org.eclipse.swt.widgets.Text;
import org.eclipse.swt.widgets.Tree;
import org.eclipse.swt.widgets.TreeItem;
import org.eclipse.swt.graphics.Point;
import org.eclipse.swt.graphics.Rectangle;
import org.eclipse.swt.widgets.MessageBox;
import org.eclipse.swt.dnd.*;
import org.eclipse.swt.layout.*;

public class LeftTabs {

// left hand "tab" tree
static Tree tree;
// right hand viewer
static CompositePane view;
// temporary TreeItem used to hold data during Drag & Drop
static TreeItem sourceTreeItem;
// keep track of number of treeItems created
static int count = 1;

public static void main(String[] args) {
// setup the display & shell & layout
final Display display = new Display();
final Shell shell = new Shell(display);
FillLayout fillLayout = new FillLayout();
// columns
fillLayout.type = SWT.HORIZONTAL;
// break 'em up a little
fillLayout.spacing = 3;
fillLayout.marginHeight = 3;
shell.setLayout(fillLayout);

// And here's the tree
tree = new Tree(shell, SWT.BORDER);
// filled with some data
for (int i = 1; i <= 3; i++) {
TreeItemData myData = new TreeItemData();
myData.Name = "Item" + i;
myData.field1 = myData.Name + " Data";
TreeItem item = new TreeItem(tree, SWT.NONE);
count++;
item.setText(myData.Name);
item.setData(myData);
}

// And here's the right hand pane
view = new CompositePane(shell);

// right click menu on the tree
tree.addListener(SWT.MenuDetect, new Listener() {
public void handleEvent(Event event) {
Menu menu = new Menu(shell, SWT.POP_UP);
// NEW ITEM
MenuItem item_new = new MenuItem(menu, SWT.PUSH);
item_new.setText("New Item");
// when click "New Item", add an item to tree
item_new.addListener(SWT.Selection, new Listener() {
public void handleEvent(Event e) {
TreeItem[] selection = tree.getSelection();
int index;
if (selection.length != 0)
index = tree.indexOf(selection[0]) + 1;
else
index = 0;
TreeItem item = new TreeItem(tree, SWT.NONE, index);
TreeItemData myData = new TreeItemData();
myData.Name = "Item" + count++;
myData.field1 = myData.Name + " Data";
item.setText(myData.Name);
item.setData(myData);
view.setData((TreeItemData) item.getData());
tree.setSelection(item);
}
}); // end item_new event

// DELETE ITEM
MenuItem item_delete = new MenuItem(menu, SWT.PUSH);
item_delete.setText("Delete Item");
// when click "Delete Item", delete selected item
item_delete.addListener(SWT.Selection, new Listener() {
public void handleEvent(Event e) {
TreeItem[] selection = tree.getSelection();
int index = tree.indexOf(selection[0]);
for (int i=0; i < selection.length; i++){
selection[i].dispose();
}
// item deleted need new selection
TreeItem[] items = tree.getItems();
if (items.length != 0) {
if (index > items.length-1) {
// it would be better to select an item closer to the deleted item?
view.setData((TreeItemData) items[items.length-1].getData());
tree.setSelection(items[items.length-1]);
} else {
view.setData((TreeItemData) items[index].getData());
tree.setSelection(items[index]);
}
}
}
}); // end item_delete

menu.setLocation(event.x, event.y);
menu.setVisible(true);
while (!menu.isDisposed() && menu.isVisible()) {
if (!display.readAndDispatch())
display.sleep();
}
menu.dispose();
}
}); // end right click menu

// when a new treeItem is selected, update the view
tree.addListener(SWT.Selection, new Listener() {
public void handleEvent(Event event)
{
view.setData((TreeItemData) event.item.getData());
}
}); // end update view with tree item data

// when the tree looses focus make sure things are in order
tree.addListener(SWT.FocusOut, new Listener() {
public void handleEvent(Event event) {
TreeItem[] selection = tree.getSelection();
// no item selected - view data is ambigous, possible data loss
if (selection.length == 0) {
MessageBox messageBox = new MessageBox(shell, SWT.ICON_ERROR);
messageBox.setText("Possible Loss of Data.");
messageBox.setMessage("Think about what you did to cause the view data and the tree selection to get out of sync, and decide how you want your program to behave!");
messageBox.open();
//*/
TreeItem[] items = tree.getItems();
if (items.length != 0) {
view.setData((TreeItemData) items[0].getData());
tree.setSelection(items[0]);
} else {
// no items in view?! you can't delete the last item...
TreeItemData myData = new TreeItemData();
myData.Name = "Item" + count++;
myData.field1 = myData.Name + " Data";
TreeItem item = new TreeItem(tree, SWT.NONE);
item.setText(myData.Name);
item.setData(myData);
view.setData((TreeItemData) item.getData());
tree.setSelection(item);
}
//*/
}
}
}); // end tree loose focus

// setup DragSource
DragSource source = new DragSource(tree, DND.DROP_COPY);
source.setTransfer(new Transfer[] { TreeItemDataTransfer.getInstance() });

source.addDragListener(new DragSourceAdapter() {
public void dragStart(DragSourceEvent event) {
TreeItem[] selection = tree.getSelection();
if (selection.length > 0 && selection[0].getData() != null) {
event.doit = true;
sourceTreeItem = selection[0];
} else {
event.doit = false;
}
} // end dargStart

public void dragSetData(DragSourceEvent event) {
if (TreeItemDataTransfer.getInstance().isSupportedType(event.dataType))
event.data = sourceTreeItem.getData();
} // end dargSetData

public void dragFinished(DragSourceEvent event) {
if (event.doit) {
sourceTreeItem.dispose();
}
sourceTreeItem = null;
} // end dragFinished
}); // end DragSource

// setup DropTarget
DropTarget target = new DropTarget(tree, DND.DROP_COPY);
target.setTransfer(new Transfer[] {TreeItemDataTransfer.getInstance() });

target.addDropListener(new DropTargetAdapter() {
public void dragEnter(DropTargetEvent event) {
event.detail = DND.DROP_COPY;
} // end dragEnter

public void dragOver(DropTargetEvent event) {
event.feedback = DND.FEEDBACK_SCROLL;
// if the drop target is a specific item in the tree
if (event.item != null) {
TreeItem item = (TreeItem) event.item;
Point pt = display.map(null, tree, event.x, event.y);
Rectangle bounds = item.getBounds();
// give visual cue of drop location to user
if (pt.y < bounds.y + bounds.height/2) {
event.feedback |= DND.FEEDBACK_INSERT_BEFORE;
} else {
event.feedback |= DND.FEEDBACK_INSERT_AFTER;
}
} else {
// set event.item to last item in list & set feedback to after
}
} // end dragOver

public void drop(DropTargetEvent event) {
try {
if (event.data == null) {
event.detail = DND.DROP_NONE;
return;
}

TreeItem newItem;
// if the dropTarget is a specific item in the tree
if (event.item != null) {
TreeItem selection = (TreeItem) event.item;
Point pt = display.map(null, tree, event.x, event.y);
Rectangle bounds = selection.getBounds();
//TreeItem[] items = tree.getItems();
// find index of selection
int index = tree.indexOf(selection);
// insert newItem at index
if (pt.y < bounds.y + bounds.height/2) {
// insert before
newItem = new TreeItem(tree, SWT.NONE, index);
} else {
// insert after
newItem = new TreeItem(tree, SWT.NONE, index+1);
}
} else {
// no specific item selected, drop at the end
newItem = new TreeItem(tree, SWT.NONE);
}

TreeItemData myType = (TreeItemData) event.data;
newItem.setText(myType.Name);
newItem.setData(myType);
// set selection otherwise view data gets out of sync
tree.setSelection(newItem);
} catch (RuntimeException e) {
e.printStackTrace();
}
} // end drop
}); // end DropTarget

// write view data back to tree - see CompositePane.Update()
tree.addListener(SWT.Arm, new Listener() {
public void handleEvent(Event event) {
tree.getSelection()[0].setData((TreeItemData) view.getData());
}
});

// wrap it up
shell.open();
while (!shell.isDisposed()) {
if (!display.readAndDispatch())
display.sleep();
}
display.dispose();
}

} // class LeftTabs

class TreeItemData {
// all the data to be held by each treeItem
String Name;
String field1;
} // new members must be explicitly handled in nativeToJava & javaToNative

// Transfer class for handling DND of TreeItemData
class TreeItemDataTransfer extends ByteArrayTransfer {
private static final String TreeItemData_TRANSFER_NAME = "TreeItemData_TRANSFER";
private static final int TreeItemData_TRANSFER_ID = registerType (TreeItemData_TRANSFER_NAME);
private static TreeItemDataTransfer instance = new TreeItemDataTransfer();

public static TreeItemDataTransfer getInstance() {
return instance;
}

protected String[] getTypeNames() {
return new String[] { TreeItemData_TRANSFER_NAME };
}

protected int[] getTypeIds() {
return new int[] {TreeItemData_TRANSFER_ID};
}

public void javaToNative (Object object, TransferData transferData) {
if (object == null || !(object instanceof TreeItemData))
return;

TreeItemData myType = (TreeItemData) object;

if (isSupportedType(transferData)) {
try {
ByteArrayOutputStream stream = new ByteArrayOutputStream();
DataOutputStream out = new DataOutputStream(stream);
// write out each member in your custom dataType
out.writeUTF(myType.Name);
out.writeUTF(myType.field1);
out.close();

super.javaToNative(stream.toByteArray(), transferData);
} catch (IOException e) {
e.printStackTrace();
}
}
} // end javaToNative

public Object nativeToJava (TransferData transferData) {
if (isSupportedType(transferData)) {
byte[] buffer = (byte[]) super.nativeToJava(transferData);
if (buffer == null)
return null;

TreeItemData myType = new TreeItemData();

try {
ByteArrayInputStream stream = new ByteArrayInputStream(buffer);
DataInputStream in = new DataInputStream(stream);
// read in each member in your custom dataType
myType.Name = in.readUTF();
myType.field1 = in.readUTF();
in.close();
} catch (IOException e) {
e.printStackTrace();
return null;
}
return myType;
} else {
return null;
}
} // end nativetoJava

} // end TreeItemDataTransfer

// Composite widget for defining the right hand "view"
class CompositePane extends Composite {

// local TreeItemData, to keep track of changes
static TreeItemData viewData;

// declarations of pane view widgets
static Text text;

public CompositePane (Composite c) {
// setup layout of the pane
super(c, SWT.NONE);
this.setLayout(new FillLayout());

// put widgets in the pane
text = new Text(this, SWT.BORDER | SWT.MULTI | SWT.WRAP);

// add a loose focus listener to EVERY *EDITABLE* widget
text.addListener(SWT.Deactivate, new Listener()
{
public void handleEvent(Event event)
{
// sync changes in view back to dataum
Update();
}
});
} // CompositePane constructor

public void setData (TreeItemData n)
{
viewData = n;
// populate the pane's elements with the viewData
text.setText(viewData.field1);
}

// update viewData, then notify listener that it should sync to the tree
public void Update() {
viewData.field1 = text.getText();
this.notifyListeners(SWT.Modify, new Event());
}

public TreeItemData getData()
{
return viewData;
}

} // class CompositePane
