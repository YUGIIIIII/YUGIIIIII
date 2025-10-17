#include <bits/stdc++.h>
using namespace std;

// Store grammar as a map: Non-terminal -> list of productions
map<string, vector<string>> grammar;

// Function to print the grammar
void printGrammar(const map<string, vector<string>> &g) {
    cout << "--------------------------------------------\n";
    for (auto &p : g) {
        cout << p.first << " -> ";
        for (size_t i = 0; i < p.second.size(); ++i) {
            cout << p.second[i];
            if (i < p.second.size() - 1) cout << " | ";
        }
        cout << endl;
    }
    cout << "--------------------------------------------\n";
}

// Function to eliminate immediate left recursion
map<string, vector<string>> eliminateLeftRecursion(const map<string, vector<string>> &g) {
    map<string, vector<string>> newGrammar;

    for (auto &p : g) {
        const string &A = p.first;      // Current non-terminal
        vector<string> alpha, beta;      // alpha: recursive parts, beta: non-recursive

        // Separate productions into alpha and beta
        for (const string &prod : p.second) {
            if (prod.substr(0, A.size()) == A)
                alpha.push_back(prod.substr(A.size()));  // remove leading A
            else
                beta.push_back(prod);
        }

        if (!alpha.empty()) {
            string Aprime = A + "'";  // New non-terminal for recursion
            for (const string &b : beta) newGrammar[A].push_back(b + Aprime);
            for (const string &a : alpha) newGrammar[Aprime].push_back(a + Aprime);
            newGrammar[Aprime].push_back("^"); // epsilon
        } else {
            newGrammar[A] = p.second; // No recursion
        }
    }
    return newGrammar;
}

// Function to perform left factoring
map<string, vector<string>> leftFactor(const map<string, vector<string>> &g) {
    map<string, vector<string>> newGrammar;

    for (auto &p : g) {
        const string &A = p.first;
        vector<string> prods = p.second;
        bool factored = false;

        sort(prods.begin(), prods.end()); // sort to find common prefixes

        for (size_t i = 0; i < prods.size(); ++i) {
            for (size_t j = i + 1; j < prods.size(); ++j) {
                size_t k = 0;
                // Find common prefix length
                while (k < prods[i].size() && k < prods[j].size() && prods[i][k] == prods[j][k])
                    ++k;

                if (k > 0) { // Common prefix found
                    string prefix = prods[i].substr(0, k);
                    string Aprime = A + "'"; 
                    vector<string> remain;

                    // Separate remaining parts
                    for (auto &x : prods)
                        if (x.substr(0, k) == prefix)
                            remain.push_back(x.substr(k).empty() ? "^" : x.substr(k));
                        else
                            newGrammar[A].push_back(x);

                    newGrammar[A].push_back(prefix + Aprime); // Add factored production
                    newGrammar[Aprime] = remain;             // Add new non-terminal
                    factored = true;
                    break;
                }
            }
            if (factored) break;
        }
        if (!factored) newGrammar[A] = prods; // No factoring needed
    }
    return newGrammar;
}

int main() {
    int n;
    cout << "Enter number of productions: ";
    cin >> n;
    cin.ignore(); // ignore leftover newline

    // Read all productions
    for (int i = 0; i < n; ++i) {
        string line;
        cout << "Enter production " << i + 1 << ": ";
        getline(cin, line);

        size_t arrow = line.find("->");
        if (arrow == string::npos) { cout << "Invalid format!\n"; return 1; }

        string lhs = line.substr(0, arrow);           // LHS non-terminal
        string rhs = line.substr(arrow + 2);          // RHS string

        stringstream ss(rhs);
        string prod;
        while (getline(ss, prod, '|'))                // Split by '|'
            grammar[lhs].push_back(prod);
    }

    cout << "\nOriginal Grammar:\n"; 
    printGrammar(grammar);

    // Eliminate left recursion
    grammar = eliminateLeftRecursion(grammar);
    cout << "\nAfter Eliminating Left Recursion:\n"; 
    printGrammar(grammar);

    // Perform left factoring
    grammar = leftFactor(grammar);
    cout << "\nAfter Left Factoring:\n"; 
    printGrammar(grammar);

    return 0;
}