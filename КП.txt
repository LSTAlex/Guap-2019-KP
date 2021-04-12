#include <iostream>
#include <iomanip> // Манипуляторы ввода-вывода (табличная печать)
#include <fstream>
#include <vector>
#include <string>
#include <algorithm>
#include <Windows.h>

using namespace std;

// Ввод строки, пробелы заменяются на подчеркивания
string input_string()
{
	string s;
	getline(cin, s);

	// Заменим все пробелы на подчеркивание
	for (size_t i = 0; i < s.size(); i++)
	{
		if (s[i] == ' ' || s[i] == '\t')
			s[i] = '_';
	}
	return s;
}

bool is_number(string s)
{
	for (size_t i = 0; i < s.size(); i++)
	{
		if (s[i] < '0' || s[i]>'9')
		{
			return false;
		}
	}
	return true;
}

int is_number1(string s)
{
	int n;
	for (size_t i = 0; i < s.size(); i++)
	{
		if (s[i] < '0' || s[i]>'9')
		{
			size_t sz;
			n = stoi(s, &sz);
		}
	}
	return n;
}

// ввод числа с проверкой, если не число, то повторный ввод
int input_int()
{

	while (1)
	{
		string s;
		getline(cin, s);
		if (!s.empty())
		{
			if (is_number(s))

			{
				size_t sz;
				int value = stoi(s, &sz);
				return value;
			}
		}

		cout << "Ошибка: ожидается число!" << endl;
	}
}
//-------------------------------------------------

struct Aeroflot
{
	string destination;
	int flight_id;
	string plane_type;

	void print() const
	{
		// setw - задает ширину поля для вывода
		cout << left;
		cout << setw(20) << destination;
		cout << setw(15) << flight_id;
		cout << setw(20) << plane_type << endl;
	}
};

class Database
{
	vector<Aeroflot> flight;
public:
	void load();
	void print();
	void append();
	void find(string destination);
	void save();
	void sort_by_destination();
	void remove(int id);
	void edit(int id);

	bool flight_id_exists(int id);
};

void menu() // ф-ция для вывода пунктов меню в консоль
{
	cout << "0. Выйти из программы" << endl;
	cout << "1. Загрузить из файла" << endl;
	cout << "2. Печать рейсов" << endl;
	cout << "3. Добавить рейс" << endl;
	cout << "4. Найти рейс по пункту назначения" << endl;
	cout << "5. Сохранить в файл" << endl;
	cout << "6. Сортировать по пункту назначения" << endl;
	cout << "7. Удалить рейс по номеру" << endl;
	cout << "8. Изменить рейс по номеру" << endl;
	cout << endl;
}

void Database::load()
{
	ifstream fin("input.txt");
	if (!fin)
	{
		cout << "Ошибка: не могу открыть файл!" << endl;
		return;
	}


	flight.clear();
	int n;
	fin >> n;

	if (!fin.good()) {
		cout << "Ошибка формата файла или пустой файл!" << endl;
		return;
	}

	if (n == 0) {
		cout << "В файле нет записей" << endl;
	}

	for (int i = 0; i < n; i++)
	{
		Aeroflot f;
		fin >> f.destination;

		string str_id;
		fin >> str_id;
		if (is_number(str_id))
		{
			size_t sz;
			f.flight_id = stoi(str_id, &sz);

			if (flight_id_exists(f.flight_id)) {
				fin >> f.plane_type;
				continue;
			}
		}
		else {
			fin >> f.plane_type;
			continue;
		}

		fin >> f.plane_type;
		flight.push_back(f);
	}

	fin.close();
}

void Database::print()
{
	cout << left;
	cout << setw(20) << "Пункт назначения";
	cout << setw(15) << "Номер рейса";
	cout << setw(20) << "Тип самолета" << endl;

	for (size_t i = 0; i < flight.size(); i++)
	{
		flight[i].print();
	}

	cout << endl;
}

bool Database::flight_id_exists(int id)
{
	// Проверим, что рейса с таким номером еще нет
	for (int i = 0; i < flight.size(); i++)
	{
		if (flight[i].flight_id == id)
		{
			return true; // рейс найден
		}
	}
	return false;
}

void Database::append()
{
	Aeroflot f;
	cout << "Номер рейса: ";
	f.flight_id = input_int();

	if (flight_id_exists(f.flight_id)) {
		cout << "Ошибка: рейс с таким номером уже есть!" << endl;
		return;
	}

	cout << "Пункт назначения: ";
	f.destination = input_string();
	cout << "Тип самолета: ";
	f.plane_type = input_string();

	flight.push_back(f);
}

void Database::find(string destination)
{
	for (const auto& f : flight)
	{
		if (f.destination == destination)
		{
			f.print();
		}
	}
}

void Database::remove(int id)
{
	auto it = flight.begin();
	for (; it != flight.end(); ++it)
	{
		if (it->flight_id == id)
			break;
	}

	if (it != flight.end())
		flight.erase(it);
	else
		cout << "Удаляемый рейс не найден!" << endl;
}

void Database::edit(int id)
{
	auto it = flight.begin();
	for (; it != flight.end(); ++it)
	{
		if (it->flight_id == id)
			break;
	}

	if (it != flight.end())
	{
		Aeroflot f;
		cout << "Пункт назначения: ";
		f.destination = input_string();

		do {

			cout << "Номер рейса: ";
			f.flight_id = input_int();
			if (flight_id_exists(f.flight_id) && f.flight_id != it->flight_id)
			{
				cout << "Ошибка: рейс с таким номером уже есть!"
					<< endl;
			}
			else
				break;
		} while (1);

		cout << "Тип самолета: ";
		f.plane_type = input_string();
		*it = f;
	}
	else
		cout << "Редактируемый рейс не найден!" << endl;
}

void Database::save()
{
	ofstream fout("input.txt");

	int n = flight.size();
	fout << n << endl;

	for (const auto& f : flight) {
		fout << f.destination << "\t" << f.flight_id << "\t" <<
			f.plane_type << endl;
	}

	fout.close();
}

bool compareByDestination(const Aeroflot& a1, const Aeroflot& a2)
{
	return a1.destination < a2.destination;
}

void Database::sort_by_destination()
{
	sort(flight.begin(), flight.end(), compareByDestination);
}

int main()
{
	setlocale(LC_ALL, "Russian");
	SetConsoleCP(1251); // Ввод с консоли в кодировке 1251
	SetConsoleOutputCP(1251);

	Database db;
	string n;
	int num;

	int choise = 0;

	do
	{
		menu();
		cin >> choise;
		cin.get();
		switch (choise)
		{
		case 1:
			db.load();
			break;
		case 2:
			db.print();
			break;
		case 3:
			db.append();
			break;
		case 4:
			cout << "Введите пункт назначения искомого рейса: ";
			//cin >> n;
			n = input_string();
			db.find(n);
			break;
		case 5:
			db.save();
			break;
		case 6:
			db.sort_by_destination();
			break;
		case 7:
			cout << "Введите номер удаляемого рейса: ";
			num = input_int();
			db.remove(num);
			break;
		case 8:
			cout << "Введите номер редактируемого рейса: ";
			int num = input_int();
			db.edit(num);
			break;
		}
	} while (choise != 0);

	return 0;
}
