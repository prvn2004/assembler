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

int register_a, register_b, memory[10000];
int stack_pointer = 10000, program_counter = 0;

void readMachineCode(char **argv)
{
    // Open the file specified in argv[2] for input in binary mode
    ifstream inputFile;
    inputFile.open(argv[2], ios::in | ios::binary);

    // Get the size of the file in bytes
    inputFile.seekg(0, ios::end);
    int fileSizeInBytes = inputFile.tellg();

    // Calculate the number of lines (each line contains 4 bytes of data)
    int numOfLines = fileSizeInBytes / 4;

    // Go to the beginning of the file
    inputFile.seekg(0, ios::beg);

    // Read each line (4 bytes) of data from the file and store it in memory
    for (int i = 0; i < numOfLines; i++)
    {
        inputFile.read((char *)&memory[i], 4);
    }

    // Close the input file
    inputFile.close();
}

string getFilename(char **argv)
{ // Extract filename from command line argument
    string inputFilename;
    for (int i = 0; i < strlen(argv[2]); i++)
    {
        if (argv[2][i] == '.')
        {
            break;
        }
        inputFilename += argv[2][i];
    }

    // Add extension to filename
    string outputFilename = inputFilename + ".txt";

    // Return output filename
    return outputFilename;
}

/**
 * Executes the instruction pointed to by program counter (pc).
 *
 * Returns 1 if HALT instruction is encountered, 0 otherwise.
 */
int executeInstruction(ofstream *outputFile)
{
    // Get instruction from memory at the address pointed by pc
    unsigned int instruction = memory[program_counter];

    // Extract oprogram_counterode and operand from the instruction
    int opcode = instruction % 256u;
    int operand = memory[program_counter] >> 8;

    // Increment program counter
    program_counter++;

    // Execute the instruction based on the opcode
    while (true)
    {
        switch (opcode)
        {
        case 0:
            register_b = register_a;
            register_a = operand;
            break;

        case 1:
            register_a += operand;
            break;

        case 2:
            register_b = register_a;
            register_a = memory[stack_pointer + operand];
            break;

        case 3:
            memory[stack_pointer + operand] = register_a;
            register_a = register_b;
            break;

        case 4:
            register_a = memory[register_a + operand];
            break;

        case 5:
            memory[register_a + operand] = register_b;
            break;

        case 6:
            register_a = register_b + register_a;
            break;

        case 7:
            register_a = register_b - register_a;
            break;

        case 8:
            register_a = register_b << register_a;
            break;

        case 9:
            register_a = register_b >> register_a;
            break;

        case 10:
            stack_pointer += operand;
            break;

        case 11:
            stack_pointer = register_a;
            register_a = register_b;
            break;

        case 12:
            register_b = register_a;
            register_a = stack_pointer;
            break;

        case 13:
            register_b = register_a;
            register_a = program_counter;
            program_counter += operand;
            break;

        case 14:
            program_counter = register_a;
            register_a = register_b;
            break;

        case 15:
            if (register_a == 0)
            {
                program_counter += operand;
            }
            break;

        case 16:
            if (register_a < 0)
            {
                program_counter += operand;
            }
            break;

        case 17:
            program_counter += operand;
            break;

        case 18:
            return 1;

        default:
            cerr << "Invalid opcode: " << opcode << endl;
            return 1;
        }

        // Write register values to output file and console
        *outputFile << "register_a: " << register_a << "\tregister_b: " << register_b << "\tstack_pointer: " << stack_pointer << endl;
        cout << "register_a: " << register_a << "\tregister_b: " << register_b << "\tstack_pointer: " << stack_pointer << endl;
        // Fetch next instruction
        instruction = memory[program_counter];
        opcode = instruction % 256u;
        operand = memory[program_counter] >> 8;

        // Increment program counter
        program_counter++;
    }
}

int main(int argc, char **argv)
{
    if (argc != 3)
    {
        cout << "Improper Number of Arguments given" << endl;
        return 0;
    }

    string filename = getFilename(argv);

    readMachineCode(argv);

    filename += ".txt";
    ofstream out;
    out.open(filename, ios::out);
    // Execute instructions until an error occurs or the program exits
while (true)
{
    // Execute next instruction
    if (executeInstruction(&out))
    {
        // Write register values to output file and console
        out << "register_a: " << register_a << "\tregister_b: " << register_b << "\tstack_pointer: " << stack_pointer << endl;
        cout << "register_a: " << register_a << "\tregister_b: " << register_b << "\tstack_pointer: " << stack_pointer << endl;

        break;
    }
}

    out.close();

    return 0;
}
