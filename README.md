#include <iostream>
#include <windows.h>
#include <fstream>
#include <string>


bool file_exists(const char* filename) {
    std::ifstream file(filename);
    return file.good();
}


bool create_or_overwrite_file(const char* filename) {
    std::ofstream file(filename, std::ios::trunc);  
    if (file) {
        file << "This is an example file with initial content.\n";  
        file.close();
        return true;
    }
    return false;
}


bool edit_file(const char* filename) {
	    std::ofstream file(filename, std::ios::trunc);  
    if (file) {
        std::cout << "Enter the new content for the file: " << std::endl;
        std::string new_content;
        std::cin.ignore();  
        std::getline(std::cin, new_content); 
        file << new_content << std::endl;  
        file.close();
        std::cout << "File content updated successfully!" << std::endl;
        return true;
    }
    return false;
}


void memory_map_file(const char* filename, DWORD& fileSize) {
    HANDLE hFile = CreateFile(
        filename,              
        GENERIC_READ,          
        0,                     
        NULL,                  
        OPEN_EXISTING,         
        FILE_ATTRIBUTE_NORMAL, 
        NULL);                 

    if (hFile == INVALID_HANDLE_VALUE) {
        std::cerr << "Error opening file: " << filename << std::endl;
        return;
    }

    fileSize = GetFileSize(hFile, NULL);
    if (fileSize == INVALID_FILE_SIZE) {
        std::cerr << "Error getting file size." << std::endl;
        CloseHandle(hFile);
        return;
    }


    HANDLE hMapping = CreateFileMapping(
        hFile,              
        NULL,               
        PAGE_READONLY,      
        0,                  
        fileSize,           
        NULL);              

    if (hMapping == NULL) {
        std::cerr << "Error creating file mapping." << std::endl;
        CloseHandle(hFile);
        return;
    }

    LPVOID pData = MapViewOfFile(
        hMapping,          
        FILE_MAP_READ,     
        0,                 
        0,                 
        fileSize);         

    if (pData == NULL) {
        std::cerr << "Error mapping file to memory." << std::endl;
        CloseHandle(hMapping);
        CloseHandle(hFile);
        return;
    }

    
    std::cout << "File contents: " << std::endl;
    std::cout << (char*)pData << std::endl; 

    UnmapViewOfFile(pData);
    CloseHandle(hMapping);
    CloseHandle(hFile);
}


void show_menu() {
    std::cout << "\nChoose an option for the file: \n";
    std::cout << "1. Create or Overwrite the file\n";
    std::cout << "2. Edit the file content\n";
    std::cout << "3. View the file contents\n";
    std::cout << "4. Exit\n";
}

int main() {
    std::string file_path;
    std::cout << "Enter the full path of the file (e.g., E:\\21\\example.txt): ";
    std::cin >> file_path;


    DWORD fileSize = 0;

    bool running = true;
    while (running) {
        show_menu();

        int choice;
        std::cout << "Enter your choice: ";
        std::cin >> choice;

        switch (choice) {
            case 1: {

                if (!file_exists(file_path.c_str())) {
                    std::cout << "File does not exist, creating file: " << file_path << std::endl;
                }
                if (create_or_overwrite_file(file_path.c_str())) {
                    std::cout << "File created/overwritten successfully." << std::endl;
                } else {
                    std::cerr << "Failed to create/overwrite the file." << std::endl;
                }
                break;
            }

            case 2: {

                if (file_exists(file_path.c_str())) {
                    edit_file(file_path.c_str());
                } else {
                    std::cerr << "File does not exist, cannot edit." << std::endl;
                }
                break;
            }

            case 3: {

                if (file_exists(file_path.c_str())) {
                    memory_map_file(file_path.c_str(), fileSize);
                } else {
                    std::cerr << "File does not exist!" << std::endl;
                }
                break;
            }

            case 4: {

                std::cout << "Exiting the program..." << std::endl;
                running = false;
                break;
            }

            default:
                std::cerr << "Invalid choice! Please select a valid option." << std::endl;
                break;
        }
    }

    return 0;
}
