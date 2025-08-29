#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <iomanip>
#include <cctype>

using namespace std;

// ------------------ User System ------------------
class UserSystem
{
private:
    string email;
    string password;
    bool is_logged_in;

    string getValue(const string &line)
    {
        size_t pos = line.find(':');
        return (pos != string::npos) ? line.substr(pos + 1) : "";
    }

public:
    UserSystem()
    {
        email = "";
        password = "";
        is_logged_in = false;
    }

    bool loadUserData()
    {
        ifstream file("user.txt");
        if (!file)
            return false;

        string line1, line2, line3;
        getline(file, line1);
        getline(file, line2);
        getline(file, line3);
        file.close();

        email = getValue(line1);
        password = getValue(line2);
        string loginStr = getValue(line3);
        is_logged_in = (loginStr == "true");

        return true;
    }

    void updateLoginStatus(bool status)
    {
        ofstream file("user.txt");
        file << "email:" << email << endl;
        file << "password:" << password << endl;
        file << "is_login:" << (status ? "true" : "false") << endl;
        file.close();
        is_logged_in = status;
    }

    void login()
    {
        const string GREEN = "\033[32m";
        const string RED = "\033[31m";
        const string RESET = "\033[0m";
        string inputEmail, inputPassword;
        cout << "Enter email: ";
        cin >> inputEmail;
        cout << "Enter password: ";
        cin >> inputPassword;

        if (inputEmail == email && inputPassword == password)
        {
            cout << GREEN << "\n? Login successful!\n" << RESET;
            updateLoginStatus(true);
        }
        else
        {
            cout << RED << "\n? Invalid credentials!\n" << RESET;
        }
    }

    void logout()
    {
        const string YELLOW = "\033[33m";
        const string RESET = "\033[0m";
        updateLoginStatus(false);
        cout << YELLOW << "?? You have been logged out.\n" << RESET;
    }

    bool isLoggedIn() const
    {
        return is_logged_in;
    }

    void showMenu()
    {
        if (is_logged_in)
        {
            cout << "[0] Logout\n";
        }
        else
        {
            cout << "[0] Login\n";
        }
    }

    void handleMenu(int choice)
    {
        if (choice == 0)
        {
            if (is_logged_in)
                logout();
            else
                login();
        }
        else
        {
            cout << "?? Invalid option.\n";
        }
    }
};

// ------------------ Library System ------------------
class LibrarySystem
{
public:
    void viewBooks()
    {
        ifstream file("books.txt");
        if (!file)
        {
            cout << "No books found.\n";
            return;
        }

        // ANSI color codes
        const string GREEN = "\033[32m";
        const string RED = "\033[31m";
        const string YELLOW = "\033[33m";
        const string RESET = "\033[0m";

        string line;
        cout << "\n?? Book List:\n";
        cout << "-------------------------------------------------------------\n";
        cout << "| ID  | Title                        | Status     | Copies |\n";
        cout << "-------------------------------------------------------------\n";

        while (getline(file, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string title = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));

            string displayStatus, color;
            if (count == 0)
            {
                displayStatus = "borrowed";
                color = YELLOW;
            }
            else
            {
                displayStatus = "available";
                color = GREEN;
            }

            cout << "| " << setw(3) << left << id << " | " << setw(28) << left << title
                 << "| " << color << setw(11) << left << displayStatus << RESET
                 << "| " << setw(6) << left << count << "|\n";
        }

        cout << "-------------------------------------------------------------\n";
        file.close();
    }

    void addBook()
    {
        const string GREEN = "\033[32m";
        const string RESET = "\033[0m";
        string bookName;
        int numCopies;
        cout << "Enter book title: ";
        cin.ignore();
        getline(cin, bookName);
        cout << "Enter number of copies: ";
        cin >> numCopies;

        // Find last book ID
        int lastId = 0;
        ifstream inFile("books.txt");
        string line;
        while (getline(inFile, line))
        {
            size_t first = line.find('|');
            if (first != string::npos)
            {
                int id = stoi(line.substr(0, first));
                if (id > lastId)
                    lastId = id;
            }
        }
        inFile.close();

        int newId = lastId + 1;
        ofstream file("books.txt", ios::app);
        file << newId << "|" << bookName << "|" << numCopies << "\n";
        file.close();

        cout << GREEN << "? Book added successfully.\n" << RESET;
    }

    void borrowBook()
    {
        const string GREEN = "\033[32m";
        const string YELLOW = "\033[33m";
        const string RED = "\033[31m";
        const string RESET = "\033[0m";
        string query;
        cout << "Enter search term for the book you want to borrow: ";
        cin.ignore();
        getline(cin, query);

        // Convert query to lowercase
        for (auto &c : query)
            c = tolower(c);

        ifstream inFile("books.txt");
        struct Book
        {
            int id;
            string name;
            int count;
        };
        vector<Book> bookList;
        string line;
        while (getline(inFile, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string name = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));
            // Convert name to lowercase for comparison
            string nameLower = name;
            for (auto &c : nameLower)
                c = tolower(c);
            if (nameLower.find(query) != string::npos)
            {
                bookList.push_back({id, name, count});
            }
        }
        inFile.close();

        if (bookList.empty())
        {
            cout << RED << "No books found matching your search.\n" << RESET;
            return;
        }

        cout << "\nSearch Results:\n";
        cout << "-------------------------------------------------------------\n";
        cout << "| ID  | Title                        | Status     | Copies |\n";
        cout << "-------------------------------------------------------------\n";
        for (const auto &book : bookList)
        {
            string displayStatus, color;
            if (book.count == 0)
            {
                displayStatus = "borrowed";
                color = YELLOW;
            }
            else
            {
                displayStatus = "available";
                color = GREEN;
            }
            cout << "| " << setw(3) << left << book.id << " | " << setw(28) << left << book.name
                 << "| " << color << setw(11) << left << displayStatus << RESET
                 << "| " << setw(6) << left << book.count << "|\n";
        }
        cout << "-------------------------------------------------------------\n";

        cout << "Enter the ID of the book to borrow (or 0 to cancel): ";
        int choiceId;
        cin >> choiceId;
        if (choiceId == 0)
            return;

        bool found = false;
        ifstream inFile2("books.txt");
        vector<Book> allBooks;
        vector<string> booksRaw;
        while (getline(inFile2, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string name = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));
            allBooks.push_back({id, name, count});
            booksRaw.push_back(line);
        }
        inFile2.close();

        vector<string> updatedBooks;
        for (const auto &book : allBooks)
        {
            if (book.id == choiceId && book.count > 0)
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count - 1));
                found = true;
            }
            else
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count));
            }
        }

        if (!found)
        {
            cout << RED << "? Book not available or already borrowed.\n" << RESET;
            return;
        }

        ofstream outFile("books.txt");
        for (const auto &book : updatedBooks)
        {
            outFile << book << endl;
        }
        outFile.close();

        cout << GREEN << "?? Book borrowed successfully.\n" << RESET;
    }

    void returnBook()
    {
        const string GREEN = "\033[32m";
        const string RED = "\033[31m";
        const string RESET = "\033[0m";
        ifstream inFile("books.txt");
        struct Book
        {
            int id;
            string name;
            int count;
        };
        vector<Book> bookList;
        vector<string> booksRaw;

        string line;
        while (getline(inFile, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string name = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));
            bookList.push_back({id, name, count});
            booksRaw.push_back(line);
        }
        inFile.close();

        // Show all books
        cout << "\nBooks you can return:\n";
        for (const auto &book : bookList)
        {
            cout << "ID: " << book.id << " | " << book.name << " (" << book.count << " copies)\n";
        }

        cout << "Enter the ID of the book to return: ";
        int choiceId;
        cin >> choiceId;

        bool found = false;
        vector<string> updatedBooks;
        for (const auto &book : bookList)
        {
            if (book.id == choiceId)
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count + 1));
                found = true;
            }
            else
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count));
            }
        }

        if (!found)
        {
            cout << RED << "? Book was not borrowed or doesn't exist.\n" << RESET;
            return;
        }

        ofstream outFile("books.txt");
        for (const auto &book : updatedBooks)
        {
            outFile << book << endl;
        }
        outFile.close();

        cout << GREEN << "?? Book returned successfully.\n" << RESET;
    }

    void searchBooks()
    {
        string query;
        cout << "Enter search term: ";
        cin.ignore();
        getline(cin, query);

        // Convert query to lowercase
        for (auto &c : query)
            c = tolower(c);

        ifstream inFile("books.txt");
        struct Book
        {
            int id;
            string name;
            int count;
        };
        vector<Book> bookList;
        string line;
        while (getline(inFile, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string name = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));
            // Convert name to lowercase for comparison
            string nameLower = name;
            for (auto &c : nameLower)
                c = tolower(c);
            if (nameLower.find(query) != string::npos)
            {
                bookList.push_back({id, name, count});
            }
        }
        inFile.close();

        if (bookList.empty())
        {
            cout << "No books found matching your search.\n";
            return;
        }

        // ANSI color codes
        const string GREEN = "\033[32m";
        const string YELLOW = "\033[33m";
        const string RESET = "\033[0m";

        cout << "\nSearch Results:\n";
        cout << "-------------------------------------------------------------\n";
        cout << "| ID  | Title                        | Status     | Copies |\n";
        cout << "-------------------------------------------------------------\n";
        for (const auto &book : bookList)
        {
            string displayStatus, color;
            if (book.count == 0)
            {
                displayStatus = "borrowed";
                color = YELLOW;
            }
            else
            {
                displayStatus = "available";
                color = GREEN;
            }
            cout << "| " << setw(3) << left << book.id << " | " << setw(28) << left << book.name
                 << "| " << color << setw(11) << left << displayStatus << RESET
                 << "| " << setw(6) << left << book.count << "|\n";
        }
        cout << "-------------------------------------------------------------\n";

        // Borrow from search results
        cout << "Enter the ID of the book to borrow (or 0 to cancel): ";
        int choiceId;
        cin >> choiceId;
        if (choiceId == 0)
            return;

        bool found = false;
        ifstream inFile2("books.txt");
        vector<Book> allBooks;
        vector<string> booksRaw;
        while (getline(inFile2, line))
        {
            size_t first = line.find('|');
            size_t second = line.find('|', first + 1);
            int id = stoi(line.substr(0, first));
            string name = line.substr(first + 1, second - first - 1);
            int count = stoi(line.substr(second + 1));
            allBooks.push_back({id, name, count});
            booksRaw.push_back(line);
        }
        inFile2.close();

        vector<string> updatedBooks;
        for (const auto &book : allBooks)
        {
            if (book.id == choiceId && book.count > 0)
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count - 1));
                found = true;
            }
            else
            {
                updatedBooks.push_back(to_string(book.id) + "|" + book.name + "|" + to_string(book.count));
            }
        }

        if (!found)
        {
            cout << "? Book not available or already borrowed.\n";
            return;
        }

        ofstream outFile("books.txt");
        for (const auto &book : updatedBooks)
        {
            outFile << book << endl;
        }
        outFile.close();

        cout << "?? Book borrowed successfully.\n";
    }

    void showMenu()
    {
        cout << "[1] View All Books\n";
        cout << "[2] Add Book\n";
        cout << "[3] Borrow Book\n";
        cout << "[4] Return Book\n";
        cout << "[5] Search Books\n";
    }

    void handleMenu(int choice)
    {
        switch (choice)
        {
        case 1:
            viewBooks();
            break;
        case 2:
            addBook();
            break;
        case 3:
            borrowBook();
            break;
        case 4:
            returnBook();
            break;
        case 5:
            searchBooks();
            break;
        default:
            cout << "?? Invalid option.\n";
        }
    }
};

// ------------------ Main Program ------------------
int main()
{
    UserSystem user;
    LibrarySystem library;

    if (!user.loadUserData())
    {
        cerr << "? Failed to load user data.\n";
        return 1;
    }

    while (true)
    {
        cout << "\n=========== Library System Menu ===========\n";
        user.showMenu();
        if (user.isLoggedIn())
        {
            library.showMenu();
        }
        cout << "[9] Exit\n";

        cout << "\nEnter your choice: ";
        int choice;
        cin >> choice;

        if (choice == 9)
        {
            cout << "?? Exiting. Goodbye!\n";
            break;
        }

        if (choice == 0)
        {
            user.handleMenu(choice);
        }
        else if (user.isLoggedIn())
        {
            library.handleMenu(choice);
        }
        else
        {
            cout << "?? Please login first to access library options.\n";
        }
    }

    return 0;
}
