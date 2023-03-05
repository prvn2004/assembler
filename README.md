#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
#include <string.h>
#include <map>
#include <set>
#include <iomanip>

using namespace std;

// Map of label names to their corresponding memory addresses
map<string, int> labelMap;

// Set of label names that have been used in the program
set<string> usedLabels;

// Set of error messages, with corresponding line numbers
set<pair<int, string>> errorSet;

// Set of warning messages
set<string> warningSet;

// Map of error codes to their corresponding error messages
map<int, string> errorCodeMap = {
    {0, "Invalid label name"},
    {1, "Incorrect instruction format"},
    {2, "Duplicate label names"},
    {3, "Invalid number"}};

// Map of instructions without operands to their corresponding opcode values
map<string, int> without_oper_map = {
    {"add", 6},
    {"sub", 7},
    {"shl", 8},
    {"shr", 9},
    {"a2sp", 11},
    {"sp2a", 12},
    {"return", 14},
    {"HALT", 18}};

// Map of instructions with operands to their corresponding opcode values
map<string, int> with_oper_map{
    {"ldc", 0},
    {"adc", 1},
    {"ldl", 2},
    {"stl", 3},
    {"ldnl", 4},
    {"stnl", 5},
    {"adj", 10},
    {"call", 13},
    {"brz", 15},
    {"brlz", 16},
    {"br", 17},
    {"SET", 19},
    {"data", 20}};

// Map of instructions with PC offsets to their corresponding opcode values
map<string, int> instWithPcoffMap{
    {"call", 13},
    {"brz", 15},
    {"brlz", 16},
    {"br", 17}};

vector<int> machineCode;

int linePos = 1;

void remove_comments(string &line)
{
    int i = 0;
    while (line[i] != '\0')
    {
        if (line[i] == ';')
        {
            line[i] = '\0';
            break;
        }
        i++;
    }
}

bool isValidLabel(string &label)
{
    for (int j = 0; j < label.length(); j++)
    {
        char c = label[j];

        if (j == 0)
        {
            if (!isalpha(c) && c != '_')
            {
                return false;
            }
        }

        if (!isalnum(c) && c != '_')
        {
            return false;
        }
    }

    return true;
}

vector<string> extract_words(string &line)
{
    int i = 0;
    while (i < line.length())
    {
        if (line[i] == ':')
        {
            line.insert(i + 1, " ");
        }
        i++;
    }

    remove_comments(line);

    stringstream ss(line);

    vector<string> words;
    string word;
    while (ss >> word)
    {
        words.push_back(word);
    }

    return words;
}

int string_to_number(std::string &s)
{
    int base = 10;
    if (s.length() > 2)
    {
        if (s[0] == '0' && s[1] == 'x')
        {
            for (int j = 2; j < s.length(); j++)
            {
                if (!('0' <= s[j] && s[j] <= '9') && !('a' <= tolower(s[j]) && tolower(s[j]) <= 'f'))
                {
                    errorSet.insert({linePos, errorCodeMap[3]});
                    return -1;
                }
            }
            base = 16;
        }
    }
    if (s.length() > 1)
    {
        if (s[0] == '0')
        {
            for (int j = 1; j < s.length(); j++)
            {
                if (!('0' <= s[j] && s[j] <= '7'))
                {
                    errorSet.insert({linePos, errorCodeMap[3]});
                    return -1;
                }
            }
            base = 8;
        }
    }
    int start_pos = 0;
    if (s[0] == '+' || s[0] == '-')
    {
        start_pos = 1;
    }
    for (int j = start_pos; j < s.length(); j++)
    {
        if (!isdigit(s[j]))
        {
            errorSet.insert({linePos, errorCodeMap[3]});
            return -1;
        }
    }
    return stoi(s, 0, base);
}

void insertMachineCode(int nop, vector<string> word, ofstream *list)
{
    int code = 0;

    if (nop == 0)
    {
        machineCode.push_back(without_oper_map[word[0]]);
        *list << setw(8) << setfill('0') << hex << without_oper_map[word[0]] << " ";
        return;
    }
    else
    {
        if (word.size() == 1)
        {
            return;
        }

        if (word[0] == "SET")
        {
            return;
        }
        if (word[0] == "data")
        {
            if (labelMap.find(word[1]) != labelMap.end())
            {
                usedLabels.insert(word[1]);
                code |= labelMap[word[1]];
            }
            else
            {
                code |= string_to_number(word[1]);
            }

            machineCode.push_back(code);
            *list << setw(8) << setfill('0') << hex << code << " ";
            return;
        }

        code = with_oper_map[word[0]];

        if (instWithPcoffMap.find(word[0]) != instWithPcoffMap.end())
        {
            if (labelMap.find(word[1]) != labelMap.end())
            {
                usedLabels.insert(word[1]);
                code |= ((labelMap[word[1]] - machineCode.size() - 1) << 8);
            }
            else
            {
                errorSet.insert({linePos, "No such label"});
            }
        }
        else
        {
            if (labelMap.find(word[1]) != labelMap.end())
            {
                usedLabels.insert(word[1]);
                code |= ((labelMap[word[1]]) << 8);
            }
            else
            {
                code |= (string_to_number(word[1]) << 8);
            }
        }
        machineCode.push_back(code);
        *list << setw(8) << setfill('0') << hex << code << " ";
    }
}

void generate_log_file(const string &filename)
{
    string log_filename = filename + ".log";
    ofstream log_file(log_filename);

    if (errorSet.size() > 0)
    {
        log_file << "ERRORS:" << endl;
    }

    for (auto &err : errorSet)
    {
        log_file << err.second << " on line " << err.first << endl;
        cout << "ERROR: " << err.second << " on line " << err.first << endl;
    }

    if (warningSet.size() > 0)
    {
        log_file << "WARNINGS:" << endl;
    }

    for (auto &warning : warningSet)
    {
        log_file << warning << endl;
        cout << "WARNING: " << warning << endl;
    }

    log_file.close();
}

void write_object_file(const string &filename)
{
    string obj_filename = filename + ".obj";

    ofstream obj_file(obj_filename, ios::out | ios::binary);

    for (int i = 0; i < machineCode.size(); i++)
    {
        obj_file.write((char *)&machineCode[i], sizeof(int));
    }

    obj_file.close();
}

int main(int argc, char **argv)
{
    if (argc != 2)
    {
        cout << "Error: Invalid number of arguments." << endl;
    }

    string filename;
    string input = argv[1];

    for (int i = 0; i < input.length(); i++)
    {
        if (input[i] == '.')
        {
            break;
        }
        filename += input[i];
    }

    ifstream inputFile;
    inputFile.open(argv[1], ios::in);

    // Check if file is open
    if (!inputFile.is_open())
    {
        // Initialize program counter and line position
        int pc = 0;
        int linePos = 1;

        // Loop through each line in the input file
        string line;
        while (getline(inputFile, line))
        {
            // Extract words from the line
            vector<string> words = extract_words(line);

            // Check if the line is empty
            if (words.empty())
            {
                linePos++;
                continue;
            }

            // Check if the first word is a label
            if (words[0].back() == ':')
            {
                string labelToCheck = words[0].substr(0, words[0].length() - 1);
                // Check if the label is valid
                if (isValidLabel(labelToCheck))
                {
                    // Check if the label already exists
                    if (labelMap.find(labelToCheck) == labelMap.end())
                    {
                        // Add the label to the label map
                        labelMap[labelToCheck] = pc;
                    }
                    else
                    {
                        // Add duplicate label error
                        errorSet.insert({linePos, errorCodeMap[2]});
                    }

                    // Check if there are any instructions after the label
                    if (words.size() == 1)
                    {
                        linePos++;
                        continue;
                    }
                    else
                    {
                        // Check if the instruction is valid
                        if (words.size() == 3)
                        {
                            if (with_oper_map.find(words[1]) == with_oper_map.end())
                            {
                                errorSet.insert({linePos, errorCodeMap[1]});
                            }

                            // Check if the instruction is SET and assign the label value
                            if (words[1] == "SET")
                            {
                                labelMap[labelToCheck] = string_to_number(words[2]);
                                linePos++;
                                continue;
                            }
                        }
                        else if (words.size() == 2)
                        {
                            if (without_oper_map.find(words[1]) == without_oper_map.end())
                            {
                                errorSet.insert({linePos, errorCodeMap[1]});
                            }
                        }
                        else
                        {
                            errorSet.insert({linePos, errorCodeMap[1]});
                        }
                    }
                }
                else
                {
                    // Add incorrect label error
                    errorSet.insert({linePos, errorCodeMap[0]});
                }
            }
            else
            {
                // Check if the instruction is valid
                if (words.size() == 2)
                {
                    if (with_oper_map.find(words[0]) == with_oper_map.end())
                    {
                        errorSet.insert({linePos, errorCodeMap[1]});
                    }
                }
                else if (words.size() == 1)
                {
                    if (without_oper_map.find(words[0]) == without_oper_map.end())
                    {
                        errorSet.insert({linePos, errorCodeMap[1]});
                    }
                }
                else
                {
                    errorSet.insert({linePos, errorCodeMap[1]});
                }
            }

            // Increment program counter and line position
            pc++;
            linePos++;
        }
    }
    else
    {
        cout << "Unable to open the file" << endl;
    }

    inputFile.close();

    string inputFileName = argv[1];
    string listFileName = inputFileName + ".lst";

    ofstream listFile;
    listFile.open(listFileName, ios::out);

    ifstream list;
    inputFile.open(inputFileName, ios::in);

    int lineNum = 1;
    string line;
    while (getline(inputFile, line))
    {
        listFile << setw(8) << setfill('0') << hex << lineNum << " ";
        vector<string> words = extract_words(line);

        if (words.empty())
        {
            listFile << endl;
            lineNum++;
            continue;
        }

        // remove label if present
        if (words[0].back() == ':')
        {
            words.erase(words.begin());
        }

        if (words.empty())
        {
            listFile << endl;
            lineNum++;
            continue;
        }

        // determine instruction type and insert machine code
        int instructionType = without_oper_map.find(words[0]) != without_oper_map.end() ? 0 : 1;
        insertMachineCode(instructionType, words, &listFile);

        // write original line to list file
        listFile << line << endl;
        lineNum++;
    }

    // check for unused labels
    for (const auto &labelPair : labelMap)
    {
        if (usedLabels.find(labelPair.first) == usedLabels.end())
        {
            string warningMessage = "Unused Label: " + labelPair.first;
            warningSet.insert(warningMessage);
        }
    }

    inputFile.close();
    listFile.close();

    generate_log_file(filename);

    generate_log_file(filename);
}
