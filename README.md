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
