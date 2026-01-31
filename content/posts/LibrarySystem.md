---
title: LibrarySystem
published: 2026-01-31T10:04:25+08:00
summary: "图书管理系统，list交叉引用"
cover:
  image: https://bucket-qjy.oss-cn-qingdao.aliyuncs.com/picture/202601300127015.png
tags: [链表]
categories: '数据结构'
draft: false
lang: ''
---



```c++
#pragma once
#include <iostream>
#include <list>
#include <string>
#include <algorithm> //find函数
#include <cctype> //toupper函数
using namespace std;

class Patron;
class CheckedOutRecord;

// 书籍类
class Book
{
public:
    Book() = default;
    Book(string title)
    {
        this->title = title;
    }

    bool operator==(const Book &other) const
    {
        return this->title==other.title;
    }
private:
    Patron *parton = nullptr;
    string title;

    // 打印书籍信息
    // friend声明不受public/private影响，放在哪都行，但统一风格可放private
    friend ostream &operator<<(ostream &os, const Book &book);
    friend ostream &operator<<(ostream &os, const CheckedOutRecord &checkedOutBook);
    friend class Library;
};

// 作者类
class Author
{
public:
    Author() = default;
    Author(string name)
    {
        this->name = name;
    }
    //重载==
    bool operator==(const Author &other) const
    {
        return this->name==other.name;
    }
    // 打印作者信息
    friend ostream &operator<<(ostream &os, const Author &author);

private:
    string name;
    list<Book> books;

    friend ostream &operator<<(ostream &os, const CheckedOutRecord &checkedOutBook);
    friend class Library;
};

// 借阅记录
class CheckedOutRecord
{
public:
    CheckedOutRecord() = default;
    CheckedOutRecord(list<Book>::iterator book, list<Author>::iterator author)
        : authorRef(author), bookRef(book)
    {
    }

    // 打印借阅信息
    friend ostream &operator<<(ostream &os, const CheckedOutRecord &checkedOutBook);

    //重载==,以便后续最后归还函数remove函数使用
    bool operator==(const CheckedOutRecord &other) const
    {
        return this->bookRef==other.bookRef && this->authorRef==other.authorRef;
    }
private:
    list<Book>::iterator bookRef;
    list<Author>::iterator authorRef;

    friend class Library;
};

// 借阅人
class Patron
{
public:
    Patron() = default;
    Patron(string m_name) : name(m_name) {}

    // 打印借阅人信息
    friend ostream &operator<<(ostream &os, const Patron &parton);

    //重载==
    bool operator==(const Patron &other) const
    {
        return this->name==other.name;
    }
private:
    list<CheckedOutRecord> books;
    string name;

    friend ostream &operator<<(ostream &os, const Book &book);
    friend class Library;
};

// 图书管理系统类
// 这里数据存储使用list交叉引用
class Library
{
public:
    Library() = default;
    // 打印图书馆所有信息
    void printLibraryInfo();
    // 添加书籍
    void addBook_interface();
    //借阅书籍
    void borrowBook_interface();
    //归还书籍
    void returnBook_interface();
    //菜单
    void menu();
private:
    list<Author> catalog['Z' + 1];
    list<Patron> people['Z' + 1];

    //判断名字是否合法
    bool isValidName(string name);
    // 添加书籍
    bool addBook(string bookName, string authorName);
    //借阅书籍
    void borrowBook(Book bookTemp,Author authorTemp,Patron partonTemp); 
};

// 打印图书馆所有信息函数实现
void Library::printLibraryInfo()
{
    for (char c = 'A'; c <= 'Z'; c++)
    {
        for (auto i : catalog[c])
        {
            cout << i;
        }
    }
    for (char c = 'A'; c <= 'Z'; c++)
    {
        for (auto i : people[c])
        {
            cout << i;
        }
    }
}

//判断名字是否合法
bool Library::isValidName(string name)
{
    if(name.empty())return false;
    for (auto i : name)
    {
        if (!isalpha(i))
        {
            return false;
        }
    }
    return true;
}

// 添加书籍函数实现
void Library::addBook_interface()
{
    string bookName, authorName;
    do
    {
        cout << "input the title of book:";
        cin >> bookName;
        cout << "input the name of author:";
        cin >> authorName;
    } while (!this->addBook(bookName, authorName));

}

// 添加书籍函数实现
bool Library::addBook(string bookName, string authorName)
{
    if(!this->isValidName(bookName)||!this->isValidName(authorName))
    {
        return false;
    }
    Author newAuthor(authorName);
    Book newBook(bookName);
    //大写查询
    char firstChar=toupper(authorName[0]);
    // 判断作者是否存在

    // 需要为 Author 类提供 == 运算符，否则 std::find 无法匹配模板参数
    list<Author>::iterator authorRef = find(this->catalog[firstChar].begin(), this->catalog[firstChar].end(), newAuthor);
    if(authorRef==this->catalog[firstChar].end())
    {
        // 作者不存在，添加作者
        newAuthor.books.push_front(newBook);
        this->catalog[firstChar].push_front(newAuthor);
    }
    else
    {
        // 添加书籍到作者的书籍列表，迭代器可用->访问
        authorRef->books.push_back(newBook);
    }
    return true;
}

//借阅书籍函数实现,参数一定是有效的
void Library::borrowBook(Book bookTemp,Author authorTemp,Patron partonTemp)
{
    list<Author>::iterator authorRef=find(this->catalog[toupper(authorTemp.name[0])].begin(),
                                            this->catalog[toupper(authorTemp.name[0])].end(),authorTemp);
    list<Book>::iterator bookRef=find(authorRef->books.begin(),authorRef->books.end(),bookTemp);
    list<Patron>::iterator partonRef=find(this->people[toupper(partonTemp.name[0])].begin(),
                                            this->people[toupper(partonTemp.name[0])].end(),partonTemp);

    //借阅人是否在系统
    if(partonRef==this->people[toupper(partonTemp.name[0])].end())
    {
        // 先把新的借阅人添加到people中
        this->people[toupper(partonTemp.name[0])].push_front(partonTemp);
        // 获取指向新添加人的迭代器
        partonRef = this->people[toupper(partonTemp.name[0])].begin();
    }
    
    // 生成借阅记录并添加到借阅人的书籍列表
    CheckedOutRecord newRecord(bookRef, authorRef);
    partonRef->books.push_front(newRecord);
    // 更新 book 的借阅人指针
    bookRef->parton = &(*partonRef);
}

void Library::borrowBook_interface()
{
    Book bookTemp;
    Author authorTemp;
    Patron partonTemp;
    list<Book>::iterator bookRef;
    list<Author>::iterator authorRef;

    while(true)
    {
        cout<<"input the name of author:";
        cin>>authorTemp.name;
        authorRef=find(this->catalog[toupper(authorTemp.name[0])].begin(),
                                            this->catalog[toupper(authorTemp.name[0])].end(),authorTemp);
        if(authorRef==this->catalog[toupper(authorTemp.name[0])].end())
        {
            cout<<"author not found,input again:"<<endl;
            continue;
        }
        else break;
    }
    while(true)
    {
        cout<<"input the title of book:";
        cin>>bookTemp.title;
        bookRef=find(authorRef->books.begin(),authorRef->books.end(),bookTemp);
        if(bookRef==authorRef->books.end())
        {
            cout<<"book not found,input again:"<<endl;
            continue;
        }
        else break;
    }
    cout<<"input the name of patron:";
    cin>>partonTemp.name;
    this->borrowBook(bookTemp,authorTemp,partonTemp);
}

//归还书籍实现
void Library::returnBook_interface()
{
    Book bookTemp;
    Author authorTemp;
    Patron partonTemp;
    list<Book>::iterator bookRef;
    list<Author>::iterator authorRef;
    list<Patron>::iterator partonRef;
    while(true)
    {
        cout<<"input the name of author:";
        cin>>authorTemp.name;
        authorRef=find(this->catalog[toupper(authorTemp.name[0])].begin(),
                                            this->catalog[toupper(authorTemp.name[0])].end(),authorTemp);
        if(authorRef==this->catalog[toupper(authorTemp.name[0])].end())
        {
            cout<<"author not found,input again:"<<endl;
            continue;
        }
        else break;
    }
    while(true)
    {
        cout<<"input the title of book:";
        cin>>bookTemp.title;
        bookRef=find(authorRef->books.begin(),authorRef->books.end(),bookTemp);
        if(bookRef==authorRef->books.end())
        {
            cout<<"book not found,input again:"<<endl;
            continue;
        }
        else break;
    }
    while(true)
    {
        cout<<"input the name of patron:";
        cin>>partonTemp.name;
        partonRef=find(this->people[toupper(partonTemp.name[0])].begin(),
                                            this->people[toupper(partonTemp.name[0])].end(),partonTemp);
        if(partonRef==this->people[toupper(partonTemp.name[0])].end())
        {
            cout<<"patron not found,input again:"<<endl;
            continue;
        }
        else break;
    }
    // 从借阅人列表中删除借阅记录
    partonRef->books.remove(CheckedOutRecord(bookRef, authorRef));
    // 更新 book 的借阅人指针
    bookRef->parton = nullptr;
}

//菜单实现
void Library::menu()
{
    int choice;
    while(true)
    {
        cout<<"1.print library info"<<endl;
        cout<<"2.add book"<<endl;
        cout<<"3.borrow book"<<endl;
        cout<<"4.return book"<<endl;
        cout<<"5.exit"<<endl;
        cout<<"input your choice:";
        if (!(cin >> choice))
        {
            cout << "Invalid input, please enter a number." << endl;
            cin.clear();
            cin.ignore(1000, '\n');
            continue;
        }
        switch(choice)
        {
            case 1:
                this->printLibraryInfo();
                break;
            case 2:
                this->addBook_interface();
                break;
            case 3:
                this->borrowBook_interface();
                break;
            case 4:
                this->returnBook_interface();
                break;
            case 5:
                cout<<"exit"<<endl;
                return;
            default:
                cout<<"invalid choice,input again:"<<endl;
                break;
        }
        system("pause");
        system("cls");
        cin.ignore();
    }
}


// 重载运算符<<
// 打印author
ostream &operator<<(ostream &os, const Author &author)
{
    os << author.name << ":" << endl;
    for (auto i : author.books)
    {
        os << i;
    }
    return os;
}

// 打印book
ostream &operator<<(ostream &os, const Book &book)
{
    os << "*" << book.title;
    if (book.parton)
    {
        // 这里要把这个友元函数声明为友元才可以访问name
        os << "-CheckedOutBy:" << book.parton->name << endl;
    }
    else
    {
        os << endl;
    }
    return os;
}

// 打印借阅人
ostream &operator<<(ostream &os, const Patron &parton)
{
    os << parton.name << " has the follow book:" << endl;
    for (auto i : parton.books)
    {
        os << i;
    }
    return os;
}

// 打印借阅记录
ostream &operator<<(ostream &os, const CheckedOutRecord &checkedOutBook)
{
    os << "* " << (*checkedOutBook.bookRef).title << "-";
    os << (*checkedOutBook.authorRef).name << endl;
    return os;
}


```



# 常见错误

1. 在对容器使用find函数时，要确保查找容器中的元素可以被正确的比较，也就说要重载==

2. 关于重载<<，有两种方式

   - 利用print函数加上operator<<
   - 直接使用operator<<

   注意的是：print要加上const

3. **迭代器可以直接使用->访问**

   