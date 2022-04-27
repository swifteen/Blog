## Qt's model/view architecture

![image-20220422210212719](/images/AdvancedQtProgramming/image-20220422210212719.png)

## Qt's model hierarchy



![image-20220422210312420](/images/AdvancedQtProgramming/image-20220422210312420.png)

## 关联Table Model中的数据到QComboBox，并过滤重复数据

```c++
void MainWindow::createComboBoxModel(QComboBox *comboBox, int column)
{
	delete comboBox->model();
	UniqueProxyModel *uniqueProxyModel = new UniqueProxyModel(column,this);
	uniqueProxyModel->setSourceModel(model);
	uniqueProxyModel->sort(column, Qt::AscendingOrder);
	comboBox->setModel(uniqueProxyModel);
	comboBox->setModelColumn(column);
}
```

## 清除QTableView中的选择

```c++
QItemSelectionModel *selectionModel = tableView->selectionModel();
selectionModel->clearSelection();
```

## QItemSelection和QItemSelectionModel进行任意行组合选择

```c++
    QItemSelection selection;
    int firstSelectedRow = -1;
    for (int row = 0; row < proxyModel->rowCount(); ++row) {
        QModelIndex index = proxyModel->index(row, Zipcode);
        
        QItemSelection rowSelection(index, index);
        selection.merge(rowSelection, QItemSelectionModel::Select);
    }
    QItemSelectionModel *selectionModel = tableView->selectionModel();
    selectionModel->clearSelection();
    selectionModel->select(selection, QItemSelectionModel::Rows|
                                      QItemSelectionModel::Select);

```

## 过滤重复

```c++
bool MySortFilterProxyModel::filterAcceptsRow(int sourceRow,
        const QModelIndex &sourceParent) const
{
    QModelIndex index = sourceModel()->index(sourceRow, Column,
                                             sourceParent);
    const QString &text = sourceModel()->data(index).toString();
    if (cache.contains(text))
        return false;
    cache << text;
    return true;
}

void QSortFilterProxyModel::setFilterRegExp(const QString &pattern);
void QSortFilterProxyModel::setFilterRegExp(const QRegExp &regExp);
```

## 排序

```c++
//make the Qt::UserRole’s data the data used for sorting
QStandardItemModel::setSortRole(Qt::UserRole);
QTableView::setSortingEnabled(true);
```

## 计算ComboBox的size

If we didn’ provide the extra space, when the user started editing an item that had a spin box or combobox editor,some of the item’s text would probably be obscured

```c++
    if (role == Qt::SizeHintRole) {
        QStyleOptionComboBox option;
        switch (index.column()) {
            case PostOffice: option.currentText = item.postOffice;
                             break;
            case County: option.currentText = item.county; break;
            case State: option.currentText = item.state; break;
            default: Q_ASSERT(false);
        }
        QFontMetrics fontMetrics(data(index, Qt::FontRole).value<QFont>());
        option.fontMetrics = fontMetrics;
        QSize size(fontMetrics.width(option.currentText),
                   fontMetrics.height());
        return qApp->style()->sizeFromContents(QStyle::CT_ComboBox,&option, size);
    }
```

