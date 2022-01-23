![DD](https://user-images.githubusercontent.com/93094605/150695952-bc7d0413-1807-4a8d-a459-6a32128adfc7.png)

 >### Create a versatile Todo app in two parts:
 >  * Item Based Model
 >  * MVC Mode
 >  ----------------

# Introduction #



>So the Goal from this App is to create the following interface that manage you tasks. It should have all the features of main application such as menues, actions and toolbar. The application must store an archive of all the pending and finished tasks.




![1](https://user-images.githubusercontent.com/93094605/150695959-99500868-fc55-4d7e-be1d-f54e4d0c83b2.png)

![2](https://user-images.githubusercontent.com/93094605/150695964-7a1698d1-e23b-412f-aa1c-8eaedc5c4e7f.png)
![3](https://user-images.githubusercontent.com/93094605/150695967-b686061f-82ca-47b4-8600-58cc4e451ae2.png)


![I](https://user-images.githubusercontent.com/93094605/150695970-6290d93f-aa13-4e2d-9cbd-8617dccc45bd.png)




WE START BY:
 ```c++
// Connect the add new task button
       connect(ui->AddNewBtn, SIGNAL(clicked()), this, SLOT(SlotAddNewTask()));
 ```


 SlotAddNewTask():
 ```c++
void todo::SlotAddNewTask() {

    // Get the line edit text
    QString taskName = ui->NewTaskLineEdit->text();
    // Get current date
    QString date = QDate::currentDate().toString();

    createNewTask(taskName, date);

}
  ```





   ```c++

void todo::createNewTask(QString taskName, QString date) {

    if(taskName!=""){
    // Get the parent widget which the widget created to be child in
    QVBoxLayout* vMainLayout = qobject_cast<QVBoxLayout*>(ui->AllNewTasksContents->layout());

    // Create Frame for the main widget container
    QFrame* Hframe = new QFrame();
    Hframe->setFrameStyle(QFrame::StyledPanel);
    // Create Horizontal Box Layout as the Frame layout and also for easily add widget inside it
    
    QHBoxLayout* newTask = new QHBoxLayout(Hframe);
    Hframe->setLayout(newTask);

    // Create Frame for the details container...
    QFrame* Vframe = new QFrame();
    QVBoxLayout* taskDetails = new QVBoxLayout(Vframe);
    Vframe->setLayout(taskDetails);

    QLabel* titlelabel = new QLabel(tr("Task #%1").arg(vMainLayout->count())); // task title
    taskDetails->addWidget(titlelabel);
    QLabel* tasklabel = new QLabel(taskName); // task name
    taskDetails->addWidget(tasklabel);
    QLabel* datelabel = new QLabel(date); // task date created
    taskDetails->addWidget(datelabel);

    // Insert the task details frame inside main task box layout
    newTask->insertWidget(0, Vframe);

    // Insert horizontal spacer in between Vframe and deleteBtn
    QSpacerItem* spacer = new QSpacerItem(100, 100, QSizePolicy::Policy::Expanding, QSizePolicy::Policy::Minimum);
    newTask->insertSpacerItem(1, spacer);

    // Insert delete button
    QPushButton* deleteBtn = new QPushButton("Delete");
    newTask->insertWidget(2, deleteBtn);
    QPushButton* done = new QPushButton("Done");
    newTask->insertWidget(2, done);

    
    // Insert into parent ui frame
    vMainLayout->insertWidget(vMainLayout->count()-1, Hframe);

 ```
 Let's reference  widgets to  specific buttons:

 ```c++
 deleteBtn->setProperty("CurrentTask", QVariant(QVariant::fromValue<QFrame*>(Hframe)));

    done->setProperty("CurrentLabel", QVariant(QVariant::fromValue<QLabel*>(tasklabel)));

    done->setProperty("CurrentTask", QVariant(QVariant::fromValue<QFrame*>(Hframe)));
  ```
  The connections of delete and done buttons:

   ```c++
 
    connect(deleteBtn, SIGNAL(clicked()), this, SLOT(SlotDeleteTask()));// Connect the delete button

    
    connect(done, SIGNAL(clicked()), this, SLOT(SlotDone()));//connect donne button
  ```
deleting tasks:

```c++
   void todo::SlotDeleteTask() {
  
    QPushButton* fromButton = (QPushButton*)sender();

    //to get the widget referenced in the property
    QVariant var;
    var = fromButton->property("CurrentTask");
    QFrame* taskHBox = qvariant_cast<QFrame*>(var);

    taskHBox->deleteLater();
    delete taskHBox;
}
  ```
  to move tasks from pending to done:
   ```c++
 oid todo::SlotDone(){
    QPushButton* fromButton = (QPushButton*)sender();

    QVariant var;
    var = fromButton->property("CurrentLabel");
    QFont f;
    QLabel* label = qvariant_cast<QLabel*>(var);
   if(label->text()!=""){
    QIcon icon(":/done.png");
    icon.pixmap(5);
   auto item =new QListWidgetItem(icon,label->text());
   f.setPointSize(15); // It cannot be 0
   item->setFont(f);
   ui->listWidget->addItem(item);


            QVariant va;
            var = fromButton->property("CurrentTask");
            QFrame* taskHBox = qvariant_cast<QFrame*>(var);

            taskHBox->deleteLater();
            delete taskHBox;
  ```
  The tasks entered to our application must remains in the app so we need to save:
  ```c++
bool connOpen(){

database = QSqlDatabase::addDatabase("QSQLITE");
database.setDatabaseName("C://Users//PC//Desktop//todo.db");
        //check connection
if(database.open()){
return true;
}else {
QMessageBox::information(this,"failed", "connection failed");
return false;
}
};
  ```
  ```c++
void connClose(){
database.close();
database.removeDatabase(QSqlDatabase::defaultConnection);
    };
  ```

  ```c++
void todo::SlotAddNewTask() {

    // Get the line edit text
    QString taskName = ui->NewTaskLineEdit->text();
    if(taskName!=""){
    // Get current date
    QString date = QDate::currentDate().toString();

    createNewTask(taskName, date);

    connOpen();
    auto query =new QSqlQuery(database);
     query->prepare("INSERT INTO pending(name,date) VALUES(?,?)");
 
    query->addBindValue(taskName);
    query->addBindValue(date);

     if(query->exec())
         QMessageBox::information(this, tr("Success"),tr(" A new task has been added"));
     else
         QMessageBox::critical(this,"error :: ",query->lastError().text());

     connClose();
}}
  ```



  ```c++
void todo::SlotDone(){
//saving
connOpen();
auto query =new QSqlQuery(database);
query->prepare("INSERT INTO done2(name) VALUES(?)");

query->addBindValue(item->text());

if(!query->exec())
QMessageBox::critical(this,"error :: ",query->lastError().text());
//deleating 
QSqlQuery qry;
qry.prepare("DELETE FROM pending WHERE name='"+item->text()+"'");

qry.exec();

connClose();
}}
  ``` 



  ```c++
void todo::on_pushButton_clicked()
{    connOpen();
     QSqlQuery query;
     query.prepare("DELETE FROM done2");
     query.exec();
     connClose();
     ui->listWidget->clear();
}
  ```

![II](https://user-images.githubusercontent.com/93094605/150695984-a953413d-152c-4f27-852f-0602750d3c05.png)


>MVC consists of three kinds of objects. The Model is the application object, the View is its scrren presenation, and the Controller define sthe way the user interface reacts to user input.the three main components of this model are:

* The model: which holds the internal data for our application. Typically either in a data structure like an array or a Database.

* The View: which offer a graphical reprsentation of the data like the ones we see such as Tree or List widgets.

* The controller: which represents the brain for the application and takes responsibility for all the user interaction.




![C](https://user-images.githubusercontent.com/93094605/150695982-e28fb663-a500-4521-b3d3-b7bf84da806d.PNG)


 ```c++
void todomvc::on_pushButton_clicked()
{
    QString text =ui->lineEdit->text();
       connOpen();
    if(ui->checkBox->isChecked()){

          QStandardItem *item2 = new QStandardItem();
          item2->setText(text);
          item2->setIcon(QIcon(":/done.png"));
          listItem << item2;

          model->appendRow(item2);

          ui->listView->setModel(model);

           }

     else {

          QStandardItem *item2 = new QStandardItem();
          item2->setText(text);
          item2->setIcon(QIcon(":/done.png"));
          listItem2 << item2;

          model3->appendRow(item2);

          ui->listView_2->setModel(model3);

}
     connClose();
}
  ```

to delete:
   ```c++
void todomvc::on_clearSelect_clicked()
{

QStringList list;
QModelIndexList indexes = ui->listView->selectionModel()->selectedIndexes();
QVariant elementSelectionne = model->data(indexes[0] ,Qt::DisplayRole);
list << elementSelectionne.toString();
while(indexes.size()) {
model->removeRow(indexes.first().row());
indexes = ui->listView->selectionModel()->selectedIndexes();
QModelIndex indexElementSelectionne = ui->listView->selectionModel()->currentIndex();
QVariant elementSelectionne = model->data(indexElementSelectionne, Qt::DisplayRole);
list << elementSelectionne.toString();
     }
 ```
to delete from the database:
 ```c++
connOpen();
       for(int i=0;i<list.size();i++){
            QSqlQuery query;
           query.prepare("DELETE FROM done WHERE name='"+list[i]+"'");
           query.exec();
       }
        connClose();
  ```
for the loading:
   ```c++
connOpen();
        auto qry =new QSqlQuery(database);
        qry->prepare("select * from done");
        qry->exec();

        int idName = qry->record().indexOf("name");
        while (qry->next())
        {
          QStandardItem *item2 = new QStandardItem();
          QString name = qry->value(idName).toString();
          item2->setText(name);
          item2->setIcon(QIcon(":/done.png"));
          model->appendRow(item2);
          //  list where all the items of the session will be saved
          listItem << item2;
        }
             ui->listView->setModel(model);
        }

 ```

to save:
 ```c++
void todomvc::saving(){

    connOpen();
    auto qry =new QSqlQuery(database);
    for(int i=0;i<listItem.size();i++){
        if(listItem[i]->text()!=""){
     qry->prepare("INSERT INTO done(name) VALUES(?)");
     qry->addBindValue(listItem[i]->text());
    }
}
    for(int i=0;i<listItem2.size();i++){
     qry->prepare("INSERT INTO tasks(name) VALUES(?)");
     qry->addBindValue(listItem2[i]->text());
    }

     qry->exec();
}
void todomvc::closeEvent(QCloseEvent *e)
{
    saving();
}
  ```
# Conclusion
The Qt Framework handles the MVC pattern implicitly, especially when we work with pre-built APIs of the model, view, and delegation classes.Combined with the trend of front-end design in recent years, the final choice is QT + Vue + element UI + SQLite (the database is selected according to the needs)

created by:
+ YASSINE SQUALLI HOUSSAINI 
+ SALAH BOUZIDI






