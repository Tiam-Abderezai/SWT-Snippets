# SWT-Snippets

import org.eclipse.swt.SWT;
import org.eclipse.swt.dnd.*;
import org.eclipse.swt.layout.*;
import org.eclipse.swt.widgets.*;

public class LeftTabsDnD {

  static class ItemData {
    String name, field1;
    ItemData(String n) { name = n; field1 = n + " Data"; }
    @Override public String toString() { return name + "\n" + field1; }
    static ItemData from(String s) {
      String[] p = s.split("\n", 2);
      ItemData d = new ItemData(p[0]);
      if (p.length > 1) d.field1 = p[1];
      return d;
    }
  }

  public static void main(String[] args) {
    Display dsp = new Display();
    Shell shell = new Shell(dsp);
    shell.setText("DnD Example");
    shell.setLayout(new GridLayout(2, false));

    // left tree
    Tree tree = new Tree(shell, SWT.BORDER | SWT.SINGLE | SWT.V_SCROLL);
    tree.setLayoutData(new GridData(SWT.FILL, SWT.FILL, true, true));
    for (int i = 1; i <= 5; i++) add(tree, new ItemData("Item" + i));

    // right editor
    Text editor = new Text(shell, SWT.BORDER | SWT.MULTI | SWT.WRAP | SWT.V_SCROLL);
    editor.setLayoutData(new GridData(SWT.FILL, SWT.FILL, true, true));

    // sync selection -> editor
    tree.addListener(SWT.Selection, e -> {
      TreeItem it = (TreeItem)e.item;
      ItemData d = (ItemData) it.getData();
      editor.setData("bound", it);
      editor.setText(d.field1);
    });
    editor.addListener(SWT.Deactivate, e -> {
      TreeItem it = (TreeItem) editor.getData("bound");
      if (it != null && !it.isDisposed())
        ((ItemData) it.getData()).field1 = editor.getText();
    });

    // Drag & Drop with TextTransfer
    Transfer[] tx = { TextTransfer.getInstance() };

    DragSource drag = new DragSource(tree, DND.DROP_MOVE | DND.DROP_COPY);
    drag.setTransfer(tx);
    drag.addDragListener(new DragSourceAdapter() {
      @Override public void dragSetData(DragSourceEvent e) {
        TreeItem[] sel = tree.getSelection();
        if (sel.length > 0)
          e.data = ((ItemData) sel[0].getData()).toString();
      }
      @Override public void dragFinished(DragSourceEvent e) {
        if (e.detail == DND.DROP_MOVE) {
          TreeItem[] sel = tree.getSelection();
          if (sel.length > 0) sel[0].dispose();
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
        int insert = tree.getItemCount();
        if (e.item instanceof TreeItem) insert = tree.indexOf((TreeItem)e.item) + 1;
        TreeItem it = new TreeItem(tree, SWT.NONE, insert);
        it.setText(d.name);
        it.setData(d);
        tree.setSelection(it);
        editor.setData("bound", it);
        editor.setText(d.field1);
      }
    });

    // select first item
    if (tree.getItemCount() > 0) {
      TreeItem it = tree.getItem(0);
      tree.setSelection(it);
      editor.setData("bound", it);
      editor.setText(((ItemData) it.getData()).field1);
    }

    shell.setSize(500, 350);
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
