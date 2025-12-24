~~~c++
/*----All_Include.h----*/

#ifndef MYCPPHOMEWORK_ALL_INCLUDE_H
#define MYCPPHOMEWORK_ALL_INCLUDE_H

#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <memory>
#include <iomanip>

using namespace std;

#include "Information.h"
#include "Emlopee.h"
#include "Technician.h"
#include "Manager.h"
#include "Salesman.h"
#include "ManagerSales.h"
#include "Info.h"
#include "Menu.h"

#endif //MYCPPHOMEWORK_ALL_INCLUDE_H

/*----main.cpp----*/

#include "./HeaderFiles/All_Include.h"

int main() {

    Menu menu;
    menu.run();
    return 0;

}
/*----Employee.h----*/

class Employee {
protected:
    static string companyName;
    int id;
    string name;
    Information info;
    double salary;
public:
    Employee(int i = 0, string n = "", Information inf = Information())
            : id(i), name(n), info(inf), salary(0.0) {}

    static void setCompanyName(string name) { companyName = name; }
    static string getCompanyName() { return companyName; }

    void setId(int i) { id = i; }
    void setName(string n) { name = n; }
    void setInfo(Information inf) { info = inf; }

    int getId() const { return id; }
    string getName() const { return name; }
    Information getInfo() const { return info; }
    double getSalary() const { return salary; }

    virtual void pay() = 0; // 纯虚函数
    virtual void showType() const = 0; // 纯虚函数

    virtual void display() const {
        cout << "公司: " << companyName << endl;
        cout << "工号: " << id << endl;
        cout << "姓名: " << name << endl;
        cout << "基本信息: ";
        info.display();
        cout << endl;
        cout << "职位: ";
        showType();
        cout << "月薪: " << fixed << setprecision(2) << salary << "元" << endl;
    }
    virtual void writeToFile(ofstream& file) const {
        file << id << " " << name << " ";
        info.writeToFile(file);
        file << salary << " ";
    }
    virtual void readFromFile(ifstream& file) {
        file >> id >> name;
        info.readFromFile(file);
        file >> salary;
    }
};
string Employee::companyName = "西安邮电小学";

#endif //MYCPPHOMEWORK_EMLOPEE_H

/*---Info.h----*/

#ifndef MYCPPHOMEWORK_INFO_H
#define MYCPPHOMEWORK_INFO_H

class Info {
private:
    vector<shared_ptr<Employee>> employees;
public:
    void addEmployee(shared_ptr<Employee> emp) {
        employees.push_back(emp);
    }

    void displayAll() const {
        cout << "\n===== 员工信息列表 =====" << endl;
        for (const auto& emp : employees) {
            emp->display();
            cout << "------------------------" << endl;
        }
    }

    void writeToFile(const string& filename) const {
        ofstream file(filename);
        if (!file) {
            cerr << "无法打开文件 " << filename << " 进行写入!" << endl;
            return;
        }

        for (const auto& emp : employees) {
            emp->writeToFile(file);
        }

        file.close();
        cout << "员工信息已成功写入文件 " << filename << endl;
    }

    void readFromFile(const string& filename) {
        ifstream file(filename);
        if (!file) {
            cerr << "无法打开文件 " << filename << " 进行读取!" << endl;
            return;
        }

        employees.clear();

        int type;
        while (file >> type) {
            shared_ptr<Employee> emp;
            switch (type) {
                case 1: emp = make_shared<Technician>(); break;
                case 2: emp = make_shared<Manager>(); break;
                case 3: emp = make_shared<Salesman>(); break;
                case 4: emp = make_shared<ManagerSales>(); break;
                default:
                    cerr << "未知的员工类型: " << type << endl;
                    continue;
            }
            emp->readFromFile(file);
            employees.push_back(emp);
        }

        file.close();
        cout << "员工信息已从文件 " << filename << " 读取" << endl;
    }
};

#endif //MYCPPHOMEWORK_INFO_H

/*---Infomation.h----*/

class Information {
private:
    int age;
    double weight;
    string gender;
public:
    Information(int a = 0, double w = 0.0, string g = "") : age(a), weight(w), gender(g) {}

    void setAge(int a) { age = a; }
    void setWeight(double w) { weight = w; }
    void setGender(string g) { gender = g; }

    int getAge() const { return age; }
    double getWeight() const { return weight; }
    string getGender() const { return gender; }

    void display() const {
        cout << "年龄: " << age << "岁, 体重: " << weight << "kg, 性别: " << gender;
    }

    void writeToFile(ofstream& file) const {
        file << age << " " << weight << " " << gender << " ";
    }

    void readFromFile(ifstream& file) {
        file >> age >> weight >> gender;
    }
};

/*---Manager.h----*/

#ifndef MYCPPHOMEWORK_MANAGER_H
#define MYCPPHOMEWORK_MANAGER_H

class Manager : virtual public Employee {
protected:
    double monthlyPay;
public:
    Manager(int i = 0, string n = "", Information inf = Information(),
            double pay = 8000.0)
            : Employee(i, n, inf), monthlyPay(pay) {}

    void setMonthlyPay(double pay) { monthlyPay = pay; }
    double getMonthlyPay() const { return monthlyPay; }

    void pay() override {
        salary = monthlyPay;
    }

    void showType() const override {
        cout << "经理类员工" << endl;
    }

    void writeToFile(ofstream& file) const override {
        file << "2 "; // 类型标识
        Employee::writeToFile(file);
        file << monthlyPay << endl;
    }

    void readFromFile(ifstream& file) override {
        Employee::readFromFile(file);
        file >> monthlyPay;
    }
};

#endif //MYCPPHOMEWORK_MANAGER_H

/*---ManagerSales.h----*/

#ifndef MYCPPHOMEWORK_MANAGERSALES_H
#define MYCPPHOMEWORK_MANAGERSALES_H

class ManagerSales : public Manager, public Salesman {
public:
    ManagerSales(int i = 0, string n = "", Information inf = Information(),
                 double s = 0.0, double pay = 5000.0, double rate = 0.005)
            : Employee(i, n, inf), Manager(i, n, inf, pay), Salesman(i, n, inf, s, rate) {}

    void pay() override {
        salary = monthlyPay + sales * salesRate;
    }

    void showType() const override {
        cout << "销售经理类员工" << endl;
    }

    void writeToFile(ofstream& file) const override {
        file << "4 "; // 类型标识
        Employee::writeToFile(file);
        file << monthlyPay << " " << sales << " " << salesRate << endl;
    }

    void readFromFile(ifstream& file) override {
        Employee::readFromFile(file);
        file >> monthlyPay >> sales >> salesRate;
    }
};

#endif //MYCPPHOMEWORK_MANAGERSALES_H


/*---Salesman.h----*/

class Salesman : virtual public Employee {
protected:
    double sales;
    double salesRate;
public:
    Salesman(int i = 0, string n = "", Information inf = Information(),
             double s = 0.0, double rate = 0.04)
            : Employee(i, n, inf), sales(s), salesRate(rate) {}

    void setSales(double s) { sales = s; }
    void setSalesRate(double rate) { salesRate = rate; }

    double getSales() const { return sales; }
    double getSalesRate() const { return salesRate; }

    void pay() override {
        salary = sales * salesRate;
    }

    void showType() const override {
        cout << "销售类员工" << endl;
    }

    void writeToFile(ofstream& file) const override {
        file << "3 "; // 类型标识
        Employee::writeToFile(file);
        file << sales << " " << salesRate << endl;
    }

    void readFromFile(ifstream& file) override {
        Employee::readFromFile(file);
        file >> sales >> salesRate;
    }
};

/*---Technician.h----*/

#ifndef MYCPPHOMEWORK_TECHNICIAN_H
#define MYCPPHOMEWORK_TECHNICIAN_H


class Technician : public Employee {
private:
    int hours;
    double hourlyRate;
public:
    Technician(int i = 0, string n = "", Information inf = Information(),
               int h = 0, double rate = 100.0)
            : Employee(i, n, inf), hours(h), hourlyRate(rate) {}

    void setHours(int h) { hours = h; }
    void setHourlyRate(double rate) { hourlyRate = rate; }

    int getHours() const { return hours; }
    double getHourlyRate() const { return hourlyRate; }

    void pay() override {
        salary = hours * hourlyRate;
    }

    void showType() const override {
        cout << "技术类员工" << endl;
    }

    void display() const override {
        Employee::display();
        cout << "工作时长: " << hours << "小时" << endl;
    }

    void writeToFile(ofstream& file) const override {
        file << "1 "; // 类型标识
        Employee::writeToFile(file);
        file << hours << " " << hourlyRate << endl;
    }

    void readFromFile(ifstream& file) override {
        Employee::readFromFile(file);
        file >> hours >> hourlyRate;
    }
};


#endif //MYCPPHOMEWORK_TECHNICIAN_H

/*---Menu.h----*/

#ifndef MYCPPHOMEWORK_MENU_H
#define MYCPPHOMEWORK_MENU_H


class Menu {
private:
    Info info;
public:
    void run() {
        int choice;
        do {
            displayMenu();
            cin >> choice;
            handleChoice(choice);
        } while (choice != 0);
    }

private:
    void displayMenu() const {
        cout << "\n===== 小型公司工资管理系统 =====" << endl;
        cout << "1. 添加员工信息" << endl;
        cout << "2. 显示所有员工信息" << endl;
        cout << "3. 保存到文件" << endl;
        cout << "4. 从文件读取" << endl;
        cout << "0. 退出系统" << endl;
        cout << "请选择操作: ";
    }

    void handleChoice(int choice) {
        switch (choice) {
            case 1: addEmployee(); break;
            case 2: displayEmployees(); break;
            case 3: saveToFile(); break;
            case 4: readFromFile(); break;
            case 0: exitSystem(); break;
            default: cout << "无效的选择，请重新输入!" << endl;
        }
    }

    void addEmployee() {
        int id, age, hours;
        string name, gender;
        double weight, sales;

        cout << "\n===== 添加员工信息 =====" << endl;
        cout << "工号: "; cin >> id;
        cout << "姓名: "; cin >> name;
        cout << "年龄: "; cin >> age;
        cout << "体重(kg): "; cin >> weight;
        cout << "性别: "; cin >> gender;

        Information info(age, weight, gender);

        cout << "选择员工类型:" << endl;
        cout << "1. 技术员  2. 经理  3. 销售员  4. 销售经理" << endl;
        cout << "请选择: ";
        int type;
        cin >> type;

        shared_ptr<Employee> emp;
        switch (type) {
            case 1:
                cout << "工作小时数: "; cin >> hours;
                emp = make_shared<Technician>(id, name, info, hours);
                break;
            case 2:
                emp = make_shared<Manager>(id, name, info);
                break;
            case 3:
                cout << "销售额: "; cin >> sales;
                emp = make_shared<Salesman>(id, name, info, sales);
                break;
            case 4:
                cout << "部门销售额: "; cin >> sales;
                emp = make_shared<ManagerSales>(id, name, info, sales);
                break;
            default:
                cout << "无效的员工类型!" << endl;
                return;
        }

        emp->pay();
        this->info.addEmployee(emp);
        cout << "员工信息添加成功!" << endl;
    }

    void displayEmployees() {
        info.displayAll();
    }

    void saveToFile() {
        info.writeToFile("employees.dat");
    }

    void readFromFile() {
        info.readFromFile("employees.dat");
    }

    void exitSystem() {
        cout << "感谢使用小型公司工资管理系统，再见!" << endl;
    }
};


#endif //MYCPPHOMEWORK_MENU_H
~~~



