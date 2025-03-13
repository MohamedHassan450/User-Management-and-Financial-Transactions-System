#include<iostream>
#include<cmath>
#include<string>
#include<iomanip>
#include<vector>
#include<cctype>
#include<fstream>
#include<cstdlib>

using namespace std;

//Principals
const string ClientFileName = "CleintRecord.txt";

const string UserFileName = "UserRecord.txt";

struct User
{
    string AccountName;
    string PinCode;
    int Permissions;
    bool MarkAccount = true;
};

struct Cleint
{
    string AccountName;
    string PinCode;
    string CleintName;
    string PhoneNumber;
    unsigned AccountBalance;
    bool MarkAccount = true;
};

enum eAccess { All = -1, ClientsList = 1, AddNewClient = 2, eDeleteClient = 4, eUpdateClients = 8, eFindClient = 16, eTransactions = 32, eManageUsers = 64 };

enum Efunctions { ShowCleint = 1, AddCleint = 2, DeleteCleint = 3, UpdateCleint = 4, FindCleint = 5, Transaction = 6, UserMenu = 7, Exit = 8 };

User UserPermission;

bool CheckPermission(eAccess Permission)
{
    if (UserPermission.Permissions == eAccess::All)
        return true;
    if ((Permission & UserPermission.Permissions) == Permission)
        return true;
    else
        return false;

}

void AccsessFaild()
{
    cout << "_________________________________________________________________" << endl;
    cout << "_________________________________________________________________" << endl;
    cout << "\t\t\tSorry,Accsess Denied :-(" << endl;
    cout << "_________________________________________________________________" << endl;
    cout << "_________________________________________________________________" << endl;

}

/////////////////////////////////////////////////////////////////

//AddNewCleint
Cleint ReadCleintsToAdd()
{
    Cleint AddCleint;

    cout << "Enter Account Name ?";
    getline(cin >> ws, AddCleint.AccountName);
    cout << "Enter Cleint Pin Code ?";
    getline(cin, AddCleint.PinCode);
    cout << "Enter Cleint Name ?";
    getline(cin, AddCleint.CleintName);
    cout << "Enter Phone Number ?";
    getline(cin, AddCleint.PhoneNumber);
    cout << "Enter Account Balance ?";
    cin >> AddCleint.AccountBalance;

    return AddCleint;
}

string ConvertCleintToString(Cleint AddCleint)
{
    string Convert = "";
    string Space = "#//#";

    Convert = Convert + AddCleint.AccountName;
    Convert = Convert + Space;
    Convert = Convert + AddCleint.PinCode;
    Convert = Convert + Space;
    Convert = Convert + AddCleint.CleintName;
    Convert = Convert + Space;
    Convert = Convert + AddCleint.PhoneNumber;
    Convert = Convert + Space;
    Convert = Convert + to_string(AddCleint.AccountBalance);

    return Convert;
}

void PutStringIntoFile(string Convert)
{
    fstream MyFile;
    MyFile.open(ClientFileName, ios::out | ios::app);

    if (MyFile.is_open())
    {
        MyFile << Convert << endl;
    }
    MyFile.close();
}

void DoAddingsFunctions()
{
    Cleint AddCleint = ReadCleintsToAdd();
    string Convert = ConvertCleintToString(AddCleint);
    PutStringIntoFile(Convert);
}

void AskToAddMoreOrNot()
{
    if (!CheckPermission(eAccess::AddNewClient))
    {
        AccsessFaild();
        return;

    }

    char YorNAdding = 'Y';

    do
    {
        cout << "Adding new cleint " << endl;
        cout << endl;

        DoAddingsFunctions();
        cout << endl;

        cout << "Do you want to add more cleint ? Y/N ?";
        cin >> YorNAdding;
        cout << endl;
        system("cls");

    } while (YorNAdding == toupper('y'));
}

///////////////////////////////////////////////////////////////

//ShowCleintList
vector<string> SplitLine(string Line)
{
    vector<string> SaveWords;
    string Space = "#//#";
    short Postion;
    string Word;

    while ((Postion = Line.find(Space)) != std::string::npos)
    {
        Word = Line.substr(0, Postion);

        if (Word != "")
        {
            SaveWords.push_back(Word);
        }
        Line.erase(0, Postion + Space.length());
    }

    if (Line != "")
    {
        SaveWords.push_back(Line);
    }

    return SaveWords;
}

Cleint CovertLineIntoRecord(string Line)
{
    vector<string> TakeFromSplit = SplitLine(Line);

    Cleint CleintConvert;

    CleintConvert.AccountName = TakeFromSplit[0];
    CleintConvert.PinCode = TakeFromSplit[1];
    CleintConvert.CleintName = TakeFromSplit[2];
    CleintConvert.PhoneNumber = TakeFromSplit[3];
    CleintConvert.AccountBalance = stod(TakeFromSplit[4]);

    return CleintConvert;
}

vector<Cleint> ReadFromFile()
{
    fstream MyFile;

    vector<Cleint> SaveCleintInVector;

    MyFile.open(ClientFileName, ios::in);

    if (MyFile.is_open())
    {
        string Line;
        Cleint CleintLine;

        while (getline(MyFile, Line))
        {
            CleintLine = CovertLineIntoRecord(Line);
            SaveCleintInVector.push_back(CleintLine);
        }
    }
    MyFile.close();

    return SaveCleintInVector;
}

void CleintWillPutInCleintList(Cleint CleintList)
{
    cout << "|" << setw(19) << left << CleintList.AccountName;
    cout << "|" << setw(19) << left << CleintList.PinCode;
    cout << "|" << setw(19) << left << CleintList.CleintName;
    cout << "|" << setw(19) << left << CleintList.PhoneNumber;
    cout << "|" << setw(20) << left << CleintList.AccountBalance << endl;
}

void CleintListFunction(vector<Cleint> SaveCleintInVector)
{
    cout << "\t\t\t\t\tCleint List \t\t" << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << endl;
    cout << setw(20) << left << "|Account Name";
    cout << setw(20) << left << "|Account Pin Code";
    cout << setw(20) << left << "|Client Name";
    cout << setw(20) << left << "|Phone Number";
    cout << setw(20) << left << "|Account Balance";
    cout << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << endl;
    for (Cleint C : SaveCleintInVector)
    {
        CleintWillPutInCleintList(C);
    }
}

void PrintCleintList()
{
    if (!CheckPermission(eAccess::ClientsList))
    {
        AccsessFaild();
        return;

    }

    Cleint CleintList;
    vector<Cleint> SaveCleintInVector = ReadFromFile();
    CleintListFunction(SaveCleintInVector);
}

///////////////////////////////////////////////////////////////

//DeleteCleint
string ReadAccountNameToDelete()
{
    string CleintAccountName;

    cout << "Enter Account Name ? ";
    getline(cin, CleintAccountName);

    return CleintAccountName;
}

bool FindAccountNameToDelete(vector<Cleint> SaveCleintInVector, Cleint& CleintWillDelete)
{
    string CleintAccountName;
    CleintAccountName = ReadAccountNameToDelete();

    for (Cleint V50 : SaveCleintInVector)
    {
        if (V50.AccountName == CleintAccountName)
        {
            CleintWillDelete = V50;
            return true;
        }
    }
    return false;
}

void PrintCleintBeforDelete(Cleint CleintWillDelete)
{
    cout << endl;
    cout << "Client information : " << endl;
    cout << endl;
    cout << "Account Name : " << CleintWillDelete.AccountName << endl;
    cout << "Pin Code : " << CleintWillDelete.PinCode << endl;
    cout << "Cleint Name : " << CleintWillDelete.CleintName << endl;
    cout << "Phone Number : " << CleintWillDelete.PhoneNumber << endl;
    cout << "Account Balance : " << CleintWillDelete.AccountBalance << endl;
    cout << endl;
}

bool MarkAccountForDelete(vector<Cleint>& SaveCleintInVector, Cleint CleintWillDelete)
{
    for (Cleint& B50 : SaveCleintInVector)
    {
        if (B50.AccountName == CleintWillDelete.AccountName)
        {
            B50.MarkAccount = false;
            return false;
        }
    }
    return true;
}

vector<Cleint> PasteInFileFalseOnly(vector<Cleint> SaveCleintInVector)
{
    fstream MyFile;

    MyFile.open(ClientFileName, ios::out);

    string Line;

    if (MyFile.is_open())
    {
        for (Cleint N50 : SaveCleintInVector)
        {
            if (N50.MarkAccount == true)
            {
                Line = ConvertCleintToString(N50);
                MyFile << Line << endl;
            }
        }
    }
    MyFile.close();

    return SaveCleintInVector;
}

bool DeleteProcess(vector<Cleint> SaveCleintInVector)
{
    char YorN;
    Cleint CleintWillDelete;

    if (FindAccountNameToDelete(SaveCleintInVector, CleintWillDelete))
    {
        PrintCleintBeforDelete(CleintWillDelete);
        cout << "Do you sure you want to delete this Account ? Y/N ?" << endl;
        cin >> YorN;
        if (YorN == 'y' || YorN == 'Y')
        {
            MarkAccountForDelete(SaveCleintInVector, CleintWillDelete);
            PasteInFileFalseOnly(SaveCleintInVector);
            cout << "Client Deleted Successfully " << endl;

            return true;
        }
        else if (YorN == 'n' || YorN == 'N')
        {
            cout << "Account not deleted " << endl;
            return false;
        }
    }
    else
    {
        cout << "Account not found" << endl;
        return false;
    }
}

void DoAllDeletsFunctions()
{
    if (!CheckPermission(eAccess::eDeleteClient))
    {
        AccsessFaild();
        return;

    }
    vector<Cleint> SaveCleintInVector = ReadFromFile();
    DeleteProcess(SaveCleintInVector);
}
//////////////////////////////////////////////////////////////

//UpdateCleint
string ReadAccountNumberForEdit()
{
    string AccountNumberForEdit;

    cout << "Enter account number to edit ? ";
    getline(cin, AccountNumberForEdit);

    return AccountNumberForEdit;
}

bool FindAccountNumberForEdit(string AccountNumberForEdit, vector<Cleint> SaveCleintInVector, Cleint& CleintForEdit)
{
    for (Cleint G50 : SaveCleintInVector)
    {
        if (G50.AccountName == AccountNumberForEdit)
        {
            CleintForEdit = G50;
            return true;
        }
    }
    return false;
}

void PrintAccountBeforEdit(Cleint CleintForEdit)
{
    cout << "Account Name : " << CleintForEdit.AccountName << endl;
    cout << "Pin Code : " << CleintForEdit.PinCode << endl;
    cout << "Cleint Name : " << CleintForEdit.CleintName << endl;
    cout << "Phone Number : " << CleintForEdit.PhoneNumber << endl;
    cout << "Account Balance : " << CleintForEdit.AccountBalance << endl;
}

Cleint Edit(Cleint CleintForEdit, vector<Cleint> SaveCleintInVector)
{
    Cleint CleintAfterEdit;

    cout << "Edit Cleint Records" << endl;
    cout << endl;

    cout << "Enter Cleint Pin Code ?";
    getline(cin >> ws, CleintForEdit.PinCode);
    cout << "Enter Cleint Name ?";
    getline(cin, CleintForEdit.CleintName);
    cout << "Enter Phone Number ?";
    getline(cin, CleintForEdit.PhoneNumber);
    cout << "Enter Account Balance ?";
    cin >> CleintForEdit.AccountBalance;

    CleintAfterEdit = CleintForEdit;
    SaveCleintInVector.push_back(CleintForEdit);

    return CleintAfterEdit;
}

string ConvertEditedAccountIntoString(Cleint CleintForEdit)
{
    string Convert = "";
    string Space = "#//#";

    Convert = Convert + CleintForEdit.AccountName;
    Convert = Convert + Space;
    Convert = Convert + CleintForEdit.PinCode;
    Convert = Convert + Space;
    Convert = Convert + CleintForEdit.CleintName;
    Convert = Convert + Space;
    Convert = Convert + CleintForEdit.PhoneNumber;
    Convert = Convert + Space;
    Convert = Convert + to_string(CleintForEdit.AccountBalance);

    return Convert;
}

bool MarkAccountBeforEdited(vector<Cleint>& SaveCleintInVector, string AccountNumberForEdit)
{
    for (Cleint& L50 : SaveCleintInVector)
    {
        if (L50.AccountName == AccountNumberForEdit)
        {
            L50.MarkAccount = false;
            return false;
        }
    }
    return true;
}

vector<Cleint> PasteInFileFalseOnly5(vector<Cleint> SaveCleintInVector)
{
    fstream MyFile;

    MyFile.open(ClientFileName, ios::out);

    string Line;

    if (MyFile.is_open())
    {
        for (Cleint N50 : SaveCleintInVector)
        {
            if (N50.MarkAccount == true)
            {
                Line = ConvertEditedAccountIntoString(N50);
                MyFile << Line << endl;
            }
        }
    }
    MyFile.close();

    return SaveCleintInVector;
}

void AddEditedToFile(string EditedLine)
{
    fstream MyFile;
    MyFile.open(ClientFileName, ios::out | ios::app);

    if (MyFile.is_open())
    {
        MyFile << EditedLine << endl;
    }
    MyFile.close();
}

void AllEditThing()
{
    if (!CheckPermission(eAccess::eUpdateClients))
    {
        AccsessFaild();
        return;

    }

    vector<Cleint> SaveCleintInVector = ReadFromFile();
    string AccountNumberForEdit;
    Cleint CleintForEdit;

    AccountNumberForEdit = ReadAccountNumberForEdit();

    if (FindAccountNumberForEdit(AccountNumberForEdit, SaveCleintInVector, CleintForEdit))
    {
        PrintCleintBeforDelete(CleintForEdit);
        Cleint CleintAfterEdit = Edit(CleintForEdit, SaveCleintInVector);
        string EditedLine = ConvertEditedAccountIntoString(CleintAfterEdit);

        MarkAccountBeforEdited(SaveCleintInVector, AccountNumberForEdit);
        PasteInFileFalseOnly5(SaveCleintInVector);
        AddEditedToFile(EditedLine);
    }
    else
    {
        cout << "Account Name (" << AccountNumberForEdit << ") not found " << endl;
    }
}

//////////////////////////////////////////////////////////////

//Find Cleint
string ReadCleintAccountNameForKnwoningIfno()
{
    string ReadAccountName;

    cout << "Enter Account Name ? ";
    cin >> ReadAccountName;

    return ReadAccountName;
}

bool FindAccountToKnowInfo(vector<Cleint> SaveCleintInVector, string ReadAccountName, Cleint& Information)
{
    for (Cleint X50 : SaveCleintInVector)
    {
        if (X50.AccountName == ReadAccountName)
        {
            Information = X50;
            return true;
        }
    }
    return false;
}

void PrintCleintIonformation6(Cleint X50)
{
    cout << endl;

    cout << "Client Information " << endl;
    cout << "_________________________________________________" << endl;
    cout << "Account Name : " << X50.AccountName << endl;
    cout << "Pin Code : " << X50.PinCode << endl;
    cout << "Cleint Name : " << X50.CleintName << endl;
    cout << "Phone Number : " << X50.PhoneNumber << endl;
    cout << "Account Balance : " << X50.AccountBalance << endl;
    cout << "_________________________________________________" << endl;
}

void FindInfo()
{
    if (!CheckPermission(eAccess::eFindClient))
    {
        AccsessFaild();
        return;

    }

    vector<Cleint> SaveCleintInVector = ReadFromFile();
    string ReadAccountName = ReadCleintAccountNameForKnwoningIfno();
    Cleint Information;
    if (FindAccountToKnowInfo(SaveCleintInVector, ReadAccountName, Information))
    {
        PrintCleintIonformation6(Information);
    }
    else
    {
        cout << "Account Name (" << ReadAccountName << ") not found " << endl;
    }
}

//////////////////////////////////////////////////////////////

// Deposit
string ReadAccountNameForDeposit()
{
    string AccountNameForDeposit;
    cout << "Enter Account Name ? ";
    cin >> AccountNameForDeposit;

    return AccountNameForDeposit;
}

int AmountToDeposit(vector<Cleint>& SaveCleintInVector, string AccountNameForDeposit, Cleint& Information)
{
    if (FindAccountToKnowInfo(SaveCleintInVector, AccountNameForDeposit, Information))
    {
        PrintCleintIonformation6(Information);
        cout << endl;
        int AmountDeposited;
        cout << "Enter Amount to Deposit ? ";
        cin >> AmountDeposited;

        return AmountDeposited;
    }
    else
    {
        cout << "Account not found :-(" << endl;
        return -1; // Indicates account not found
    }
}

void DepositAmount(vector<Cleint>& SaveCleintInVector, string AccountNameForDeposit, int AmountDeposited)
{
    for (Cleint& C4 : SaveCleintInVector)
    {
        if (AccountNameForDeposit == C4.AccountName)
        {
            C4.AccountBalance += AmountDeposited;
        }
    }
}

Cleint PushInVector(Cleint& Information, string AccountNameForDeposit, vector<Cleint>& SaveCleintInVector)
{
    for (Cleint& C5 : SaveCleintInVector)
    {
        if (C5.AccountName == AccountNameForDeposit)
        {
            Information = C5;
            C5.MarkAccount = false;
        }
    }
    return Information;
}

void Deposit()
{
    vector<Cleint> SaveCleintInVector = ReadFromFile();
    string AccountNameForDeposit = ReadAccountNameForDeposit();
    Cleint Information;
    int AmountDeposited = AmountToDeposit(SaveCleintInVector, AccountNameForDeposit, Information);

    if (AmountDeposited > 0)
    {
        DepositAmount(SaveCleintInVector, AccountNameForDeposit, AmountDeposited);
        Information = PushInVector(Information, AccountNameForDeposit, SaveCleintInVector);
        SaveCleintInVector.push_back(Information);
        PasteInFileFalseOnly5(SaveCleintInVector);
        cout << endl;
        cout << "Deposit Done successful." << endl;
        cout << endl;

    }
}

//////////////////////////////////////////////////////////////

// Withdraw
string ReadAccountNameForWithdraw()
{
    string AccountNameForWithdraw;
    cout << "Enter Account Name ? ";
    cin >> AccountNameForWithdraw;

    return AccountNameForWithdraw;
}

int AmountToWithdraw(vector<Cleint>& SaveCleintInVector, string AccountNameForWithdraw, Cleint& Information)
{
    if (FindAccountToKnowInfo(SaveCleintInVector, AccountNameForWithdraw, Information))
    {
        PrintCleintIonformation6(Information);
        cout << endl;
        int AmountWillWithdraw;
        cout << "Enter Amount to Withdraw ? ";
        cin >> AmountWillWithdraw;

        return AmountWillWithdraw;
    }
    else
    {
        cout << endl;
        cout << "Account not found :-(" << endl;
        cout << endl;
        return -1; // Indicates account not found
    }
}

void WithdrawAmount(vector<Cleint>& SaveCleintInVector, string AccountNameForWithdraw, int AmountWillWithdraw)
{
    for (Cleint& C4 : SaveCleintInVector)
    {
        if (AccountNameForWithdraw == C4.AccountName)
        {
            C4.AccountBalance -= AmountWillWithdraw;
        }
    }
}

Cleint PushInVectorWithdraw(Cleint& Information, string AccountNameForWithdraw, vector<Cleint>& SaveCleintInVector)
{
    for (Cleint& C5 : SaveCleintInVector)
    {
        if (C5.AccountName == AccountNameForWithdraw)
        {
            Information = C5;
            C5.MarkAccount = false;
        }
    }
    return Information;
}

void Withdraw()
{
    vector<Cleint> SaveCleintInVector = ReadFromFile();
    string AccountNameForWithdraw = ReadAccountNameForWithdraw();
    Cleint Information;
    int AmountWillWithdraw = AmountToWithdraw(SaveCleintInVector, AccountNameForWithdraw, Information);

    if (AmountWillWithdraw > 0)
    {
        WithdrawAmount(SaveCleintInVector, AccountNameForWithdraw, AmountWillWithdraw);
        Information = PushInVectorWithdraw(Information, AccountNameForWithdraw, SaveCleintInVector);
        SaveCleintInVector.push_back(Information);
        PasteInFileFalseOnly5(SaveCleintInVector);
        cout << endl;
        cout << "Withdrawal Done successful." << endl;
        cout << endl;

    }
}


//////////////////////////////////////////////////////////////

//Total Balance

int CalculateTotalAccountBalance(vector<Cleint> SaveCleintInVector)
{
    int Counter = 0;
    for (Cleint B50 : SaveCleintInVector)
    {
        Counter = Counter + B50.AccountBalance;
    }

    return Counter;
}

void PrintTotalAccountBalanceMenu(int TotalBalance)
{
    PrintCleintList();
    cout << endl;
    cout << endl;
    cout << "\t\t\t\t\t\tTotal Balances = " << TotalBalance << endl;
}

void TotalBalancesMenu()
{
    vector<Cleint> SaveCleintInVector = ReadFromFile();
    int TotalBalance = CalculateTotalAccountBalance(SaveCleintInVector);
    PrintTotalAccountBalanceMenu(TotalBalance);
}

//////////////////////////////////////////////////////////////

//Conect Transaction Menu
void ReadInTransaction()
{

    if (!CheckPermission(eAccess::eTransactions))
    {
        AccsessFaild();
        return;

    }

    system("cls");
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t\t  Transaction Menu\t\t\t" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t" << "[1]Deposit." << endl;
    cout << "\t" << "[2]Withdraw." << endl;
    cout << "\t" << "[3]Total Balance." << endl;
    cout << "\t" << "[4]Main Menu." << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;

    int choice;
    cout << "Enter your choice: ";
    cin >> choice;
    system("cls");

    switch (choice)
    {
    case 1:
        Deposit();
        system("pause");
        system("cls");
        ReadInTransaction();
        break;
    case 2:
        Withdraw();
        system("pause");
        system("cls");
        ReadInTransaction();
        break;
    case 3:
        TotalBalancesMenu();
        system("pause");
        system("cls");
        ReadInTransaction();
        break;
    case 4:
        cout << "Transaction ended :-)" << endl;
        cout << endl;
        break;
    default:
        cout << "Invalid choice" << endl;
        cout << endl;

    }
    system("pause");
    system("cls");

}


//////////////////////////////////////////////////////////////

// Show User List
vector<string> SplitLineForUser(string Line)
{
    vector<string> SaveWords;
    string Space = "#//#";
    short Position;
    string Word;

    while ((Position = Line.find(Space)) != std::string::npos)
    {
        Word = Line.substr(0, Position);

        if (Word != "")
        {
            SaveWords.push_back(Word);
        }
        Line.erase(0, Position + Space.length());
    }

    if (Line != "")
    {
        SaveWords.push_back(Line);
    }

    return SaveWords;
}

User ConvertLineIntoRecordUser(string Line)
{
    vector<string> TakeFromSplit = SplitLineForUser(Line);

    User UserConvert;

    UserConvert.AccountName = TakeFromSplit[0];
    UserConvert.PinCode = TakeFromSplit[1];
    UserConvert.Permissions = stoi(TakeFromSplit[2]);

    return UserConvert;
}

vector<User> ReadFromFileUser()
{
    fstream MyFile;

    vector<User> SaveUserInVector;

    MyFile.open(UserFileName, ios::in);

    if (MyFile.is_open())
    {
        string Line;
        User UserLine;

        while (getline(MyFile, Line))
        {
            UserLine = ConvertLineIntoRecordUser(Line);
            SaveUserInVector.push_back(UserLine);
        }
    }
    MyFile.close();

    return SaveUserInVector;
}

void UserWillPutInUserList(User UserList)
{
    cout << "|" << setw(19) << left << UserList.AccountName;
    cout << "|" << setw(19) << left << UserList.PinCode;
    cout << "|" << setw(19) << left << UserList.Permissions << endl;

}

void UserListFunction(vector<User> SaveUserInVector)
{
    cout << "\t\t\t\t\tUser List \t\t" << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << endl;
    cout << setw(20) << left << "|Account Name";
    cout << setw(20) << left << "|Pin Code";
    cout << setw(20) << left << "|Permissions";
    cout << endl;
    cout << "------------------------------------------------------------------------------------------------------" << endl;
    cout << endl;
    for (User U : SaveUserInVector)
    {
        UserWillPutInUserList(U);
    }
    cout << endl;
    cout << endl;

}

void PrintUserList()
{
    User UserList;
    vector<User> SaveUserInVector = ReadFromFileUser();
    UserListFunction(SaveUserInVector);
}

//////////////////////////////////////////////////////////////

// Add New User

User ReadUserToAdd()
{
    User AddUser;
    char YorN;

    cout << "Enter Account Name: ";
    getline(cin >> ws, AddUser.AccountName);
    cout << "Enter User Pin Code: ";
    getline(cin, AddUser.PinCode);
    cout << "Permissions: " << endl;
    cout << endl;
    cout << "Do you want to give all permissions? Y/N: ";
    cin >> YorN;
    cout << endl;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions = All;
        return AddUser;
    }

    AddUser.Permissions = 0; // Initialize to no permissions

    cout << "Do you want to give permission for Client List? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= ClientsList;
    }

    cout << "Do you want to give permission for Add New Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= AddNewClient;
    }

    cout << "Do you want to give permission for Delete Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= eDeleteClient;
    }

    cout << "Do you want to give permission for Update Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= eUpdateClients;
    }

    cout << "Do you want to give permission for Find Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= eFindClient;
    }

    cout << "Do you want to give permission for Transactions? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= eTransactions;
    }

    cout << "Do you want to give permission for Manage Users? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        AddUser.Permissions |= eManageUsers;
    }

    return AddUser;
}

string ConvertUserToStringAdd(User AddUser)
{
    string Convert = "";
    string Space = "#//#";

    Convert = Convert + AddUser.AccountName;
    Convert = Convert + Space;
    Convert = Convert + AddUser.PinCode;
    Convert = Convert + Space;
    Convert = Convert + to_string(AddUser.Permissions);

    return Convert;
}

void PutStringIntoFileUser(string Convert)
{
    fstream MyFile;
    MyFile.open(UserFileName, ios::out | ios::app);

    if (MyFile.is_open())
    {
        MyFile << Convert << endl;
    }
    MyFile.close();
}

void DoAddingsFunctionsUser()
{
    User AddUser = ReadUserToAdd();
    string Convert = ConvertUserToStringAdd(AddUser);
    PutStringIntoFileUser(Convert);
}

void AskToAddMoreUsers()
{
    char YorNAdding = 'Y';

    do
    {
        cout << "Adding new User" << endl;
        cout << endl;

        DoAddingsFunctionsUser();
        cout << endl;

        cout << "Do you want to add more Users? Y/N: ";
        cin >> YorNAdding;
        cout << endl;


    } while (toupper(YorNAdding) == 'Y');
}

//////////////////////////////////////////////////////////////

// Delete User
string ReadAccountNameToDeleteUser()
{
    string UserAccountName;

    cout << "Enter Account Name: ";
    getline(cin >> ws, UserAccountName);

    return UserAccountName;
}

bool FindAccountNameToDeleteUser(vector<User> SaveUserInVector, User& UserWillDelete)
{
    string UserAccountName;
    UserAccountName = ReadAccountNameToDeleteUser();

    for (User U : SaveUserInVector)
    {
        if (U.AccountName == UserAccountName)
        {
            UserWillDelete = U;
            return true;
        }
    }
    return false;
}

void PrintUserBeforeDelete(User UserWillDelete)
{
    cout << endl;
    cout << "User information: " << endl;
    cout << endl;
    cout << "Account Name: " << UserWillDelete.AccountName << endl;
    cout << "Pin Code: " << UserWillDelete.PinCode << endl;
    cout << "Permissions: " << UserWillDelete.Permissions << endl;
    cout << endl;
}

bool MarkAccountForDeleteUser(vector<User>& SaveUserInVector, User UserWillDelete)
{
    for (User& U : SaveUserInVector)
    {
        if (U.AccountName == UserWillDelete.AccountName)
        {
            U.MarkAccount = false;
            return false;
        }
    }
    return true;
}

vector<User> PasteInUserFileFalseOnly(vector<User> SaveUserInVector)
{
    fstream MyFile;

    MyFile.open(UserFileName, ios::out);

    string Line;

    if (MyFile.is_open())
    {
        for (User U : SaveUserInVector)
        {
            if (U.MarkAccount == true)
            {
                Line = ConvertUserToStringAdd(U);
                MyFile << Line << endl;
            }
        }
    }
    MyFile.close();

    return SaveUserInVector;
}

bool DeleteUserProcess(vector<User> SaveUserInVector)
{
    char YorN;
    User UserWillDelete;

    if (FindAccountNameToDeleteUser(SaveUserInVector, UserWillDelete))
    {
        PrintUserBeforeDelete(UserWillDelete);
        cout << "Are you sure you want to delete this User? Y/N: " << endl;
        cin >> YorN;
        if (YorN == 'y' || YorN == 'Y')
        {
            MarkAccountForDeleteUser(SaveUserInVector, UserWillDelete);
            PasteInUserFileFalseOnly(SaveUserInVector);
            cout << endl;
            cout << "User Deleted Successfully" << endl;
            cout << endl;

            return true;
        }
        else if (YorN == 'n' || YorN == 'N')
        {
            cout << "User not deleted" << endl;
            return false;
        }
    }
    else
    {
        cout << "User not found" << endl;
        return false;
    }
}

void DeleteUser()
{
    vector<User> SaveUserInVector = ReadFromFileUser();
    DeleteUserProcess(SaveUserInVector);
}

//////////////////////////////////////////////////////////////

// Find User
string ReadUserAccountNameForKnowingInfo()
{
    string ReadUserName;

    cout << "Enter Account Name: ";
    cin >> ReadUserName;

    return ReadUserName;
}

bool FindAccountToKnowUserInfo(vector<User> SaveUserInVector, string ReadAccountName, User& Information)
{
    for (User U : SaveUserInVector)
    {
        if (U.AccountName == ReadAccountName)
        {
            Information = U;
            return true;
        }
    }
    return false;
}

void PrintUserInformation(User U)
{
    cout << endl;

    cout << "User Information " << endl;
    cout << "_________________________________________________" << endl;
    cout << "Account Name: " << U.AccountName << endl;
    cout << "Pin Code: " << U.PinCode << endl;
    cout << "Permissions: " << U.Permissions << endl;
    cout << "_________________________________________________" << endl;
}

void FindUserInfo()
{
    vector<User> SaveUserInVector = ReadFromFileUser();
    string ReadAccountName = ReadUserAccountNameForKnowingInfo();
    User Information;
    if (FindAccountToKnowUserInfo(SaveUserInVector, ReadAccountName, Information))
    {
        PrintUserInformation(Information);
        cout << endl;

    }
    else
    {
        cout << endl;
        cout << "Account Name (" << ReadAccountName << ") not found" << endl;
        cout << endl;

    }
}

//////////////////////////////////////////////////////////////

// Update User
string ReadAccountNumberForEditUser()
{
    string AccountNumberForEdit;
    cout << "Enter account number to edit: ";
    getline(cin >> ws, AccountNumberForEdit);
    return AccountNumberForEdit;
}

bool FindAccountNumberForEditUser(string AccountNumberForEdit, vector<User> SaveUserInVector, User& UserForEdit)
{
    for (User U : SaveUserInVector)
    {
        if (U.AccountName == AccountNumberForEdit)
        {
            UserForEdit = U;
            return true;
        }
    }
    return false;
}

void PrintAccountBeforeEditUser(User UserForEdit)
{
    cout << "Account Name: " << UserForEdit.AccountName << endl;
    cout << "Pin Code: " << UserForEdit.PinCode << endl;
    cout << "Permissions: " << UserForEdit.Permissions << endl;
}

User EditUser(User UserForEdit, vector<User> SaveUserInVector)
{
    User UserAfterEdit;
    char YorN;
    cout << "Edit User Records" << endl;
    cout << endl;
    cout << "Enter User Pin Code: ";
    getline(cin >> ws, UserForEdit.PinCode);
    cout << endl;

    cout << "Permissions: " << endl;
    cout << endl;
    cout << "Do you want to give all permissions? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions = All;
        UserAfterEdit = UserForEdit;
        SaveUserInVector.push_back(UserForEdit);
        return UserAfterEdit;
    }

    cout << endl;

    UserAfterEdit.Permissions = 0; // Initialize to no permissions

    cout << "Do you want to give permission for Client List? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= ClientsList;
    }

    cout << "Do you want to give permission for Add New Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= AddNewClient;
    }

    cout << "Do you want to give permission for Delete Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= eDeleteClient;
    }

    cout << "Do you want to give permission for Update Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= eUpdateClients;
    }

    cout << "Do you want to give permission for Find Client? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= eFindClient;
    }

    cout << "Do you want to give permission for Transactions? Y/N: ";
    cin >> YorN;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= eTransactions;
    }

    cout << "Do you want to give permission for Manage Users? Y/N: ";
    cin >> YorN;
    cout << endl;
    if (YorN == 'y' || YorN == 'Y')
    {
        UserAfterEdit.Permissions |= eManageUsers;
    }

    UserAfterEdit = UserForEdit;
    SaveUserInVector.push_back(UserForEdit);
    return UserAfterEdit;
}

string ConvertEditedAccountIntoStringUser(User UserForEdit)
{
    string Convert = "";
    string Space = "#//#";
    Convert = Convert + UserForEdit.AccountName;
    Convert = Convert + Space;
    Convert = Convert + UserForEdit.PinCode;
    Convert = Convert + Space;
    Convert = Convert + to_string(UserForEdit.Permissions);
    return Convert;
}

bool MarkAccountBeforeEditedUser(vector<User>& SaveUserInVector, string AccountNumberForEdit)
{
    for (User& U : SaveUserInVector)
    {
        if (U.AccountName == AccountNumberForEdit)
        {
            U.MarkAccount = false;
            return false;
        }
    }
    return true;
}

vector<User> PasteInFileFalseOnlyUser(vector<User> SaveUserInVector)
{
    fstream MyFile;
    MyFile.open(UserFileName, ios::out);
    string Line;
    if (MyFile.is_open())
    {
        for (User U : SaveUserInVector)
        {
            if (U.MarkAccount == true)
            {
                Line = ConvertEditedAccountIntoStringUser(U);
                MyFile << Line << endl;
            }
        }
    }
    MyFile.close();
    return SaveUserInVector;
}

void AddEditedToFileUser(string EditedLine)
{
    fstream MyFile;
    MyFile.open(UserFileName, ios::out | ios::app);
    if (MyFile.is_open())
    {
        MyFile << EditedLine << endl;
    }
    MyFile.close();
}

void AllEditThingUser()
{
    vector<User> SaveUserInVector = ReadFromFileUser();
    string AccountNumberForEdit = ReadAccountNumberForEditUser();
    User UserForEdit;
    if (FindAccountNumberForEditUser(AccountNumberForEdit, SaveUserInVector, UserForEdit))
    {
        cout << endl;
        PrintAccountBeforeEditUser(UserForEdit);
        User UserAfterEdit = EditUser(UserForEdit, SaveUserInVector);
        string EditedLine = ConvertEditedAccountIntoStringUser(UserAfterEdit);
        MarkAccountBeforeEditedUser(SaveUserInVector, AccountNumberForEdit);
        PasteInFileFalseOnlyUser(SaveUserInVector);
        AddEditedToFileUser(EditedLine);
    }
    else
    {
        cout << endl;
        cout << "Account Name (" << AccountNumberForEdit << ") not found" << endl;
        cout << endl;
    }
}

//////////////////////////////////////////////////////////////

// Connect User Menu
void ReadInTransactionUser()
{
    if (!CheckPermission(eAccess::eManageUsers))
    {
        AccsessFaild();
        return;

    }

    system("cls");
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t\t  User Menu\t\t\t" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t" << "[1]Show User List." << endl;
    cout << "\t" << "[2]Add New User." << endl;
    cout << "\t" << "[3]Delete User." << endl;
    cout << "\t" << "[4]Update User Info." << endl;
    cout << "\t" << "[5]Find User." << endl;
    cout << "\t" << "[6]Main Menu." << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    int choice;
    cout << "Enter your choice: ";
    cin >> choice;

    system("cls");
    switch (choice)
    {
    case 1:
        PrintUserList();
        system("pause");
        system("cls");
        ReadInTransactionUser();
        break;
    case 2:
        AskToAddMoreUsers();
        system("pause");
        system("cls");
        ReadInTransactionUser();
        break;
    case 3:
        DeleteUser();
        system("pause");
        system("cls");
        ReadInTransactionUser();
        break;
    case 4:
        AllEditThingUser();
        system("pause");
        system("cls");
        ReadInTransactionUser();
        break;
    case 5:
        FindUserInfo();
        system("pause");
        system("cls");
        ReadInTransactionUser();
        break;
    case 6:
        cout << "User Menu ended :-)" << endl;
        cout << endl;
        break;
    default:
        cout << "Invalid choice" << endl;
        cout << endl;
    }
    system("pause");
    system("cls");
}

//////////////////////////////////////////////////////////////

//ConectAll

int ReadNumberToChooseSystem()
{
    short Number6 = 0;
    cout << "Enter Number Between (1) to (8) ? " << endl;
    cin >> Number6;
    cin.ignore();

    return Number6;
}

void PrintMenu();
void GoBackToMainmenu(); //Here is the Trik

void IFChoose(Efunctions EqualReadNumber)
{
    switch (EqualReadNumber)
    {
    case Efunctions::ShowCleint:
        system("cls");
        PrintCleintList();
        GoBackToMainmenu();
        break;

    case Efunctions::AddCleint:
        system("cls");
        AskToAddMoreOrNot();
        GoBackToMainmenu();
        break;

    case Efunctions::DeleteCleint:
        system("cls");
        DoAllDeletsFunctions();
        GoBackToMainmenu();
        break;

    case Efunctions::UpdateCleint:
        system("cls");
        AllEditThing();
        GoBackToMainmenu();
        break;

    case Efunctions::FindCleint:
        system("cls");
        FindInfo();
        GoBackToMainmenu();
        break;

    case Efunctions::Transaction:
        system("cls");
        ReadInTransaction();
        GoBackToMainmenu();
        break;
    case Efunctions::UserMenu:
        system("cls");
        ReadInTransactionUser();
        GoBackToMainmenu();
        break;

    case Efunctions::Exit:
        system("cls");
        cout << "----------------------------" << endl;
        cout << "Program ended :-)" << endl;
        cout << "----------------------------" << endl;
        break;
    }
}

void PrintMenu()
{
    system("cls");
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t\t  Main Menu\t\t\t" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;
    cout << "\t" << "[1]Show Cleint List." << endl;
    cout << "\t" << "[2]Add New Cleint." << endl;
    cout << "\t" << "[3]Delete Cleint." << endl;
    cout << "\t" << "[4]Update Cleint Info." << endl;
    cout << "\t" << "[5]Find Cleint." << endl;
    cout << "\t" << "[6]Transaction." << endl;
    cout << "\t" << "[7]User Menu." << endl;
    cout << "\t" << "[8]Exit." << endl;
    cout << "------------------------------------------------" << endl;
    cout << "------------------------------------------------" << endl;

    IFChoose((Efunctions)ReadNumberToChooseSystem());
}

void GoBackToMainmenu()
{
    cout << endl;
    cout << "Press any key to go to main menu.........";
    system("pause>0");
    PrintMenu();
}

//////////////////////////////////////////////////////////////

string ReadUserNameToLogin()
{

    string UserNameToLogin;
    cout << "Enter User Account Name ? ";
    cin >> UserNameToLogin;

    return UserNameToLogin;

}

string ReadUserPinCodeToLogin()
{

    string UserPinCodeToLogin;
    cout << "Enter User Account Pin Code ? ";
    cin >> UserPinCodeToLogin;

    return UserPinCodeToLogin;

}

bool CheckUserToLogin(vector<User> SaveUserInVector, string UserNameToLogin, string UserPinCodeToLogin)
{
    for (User G300 : SaveUserInVector)
    {
        if (G300.AccountName == UserNameToLogin && G300.PinCode == UserPinCodeToLogin)
        {
            UserPermission = G300;
            return true;
        }

    }
    return false;
}

void LoginMenu()
{
    system("cls");
    cout << "_______________________________________________________" << endl;
    cout << "_______________________________________________________" << endl;
    cout << "\t\t\tLogin" << endl;
    cout << "_______________________________________________________" << endl;
    cout << "_______________________________________________________" << endl;

    vector<User> SaveUserInVector = ReadFromFileUser();
    string UserNameToLogin = ReadUserNameToLogin();
    string UserPinCodeToLogin = ReadUserPinCodeToLogin();
    bool CheckUser = CheckUserToLogin(SaveUserInVector, UserNameToLogin, UserPinCodeToLogin);

    if (CheckUser)
    {
        PrintMenu();
    }
    else
    {
        LoginMenu();
    }
}


int main()
{
    LoginMenu();
}

