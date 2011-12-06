// winnergen.cpp : Defines the entry point for the console application.
//
#include <ctime>
#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <Windows.h>
#include <math.h>

#include "stdafx.h"

#define THREADS 1
typedef struct{
	int rank[35];		//ranking of each contest
	bool pickedA[35];	//true if player picked the A team to win.
	char name[20];		//name of player

	//counters for placement in each outcome (one for each thread)
	int first[THREADS];
	int second[THREADS];
	int third[THREADS];
	int fourth[THREADS];
} Player;

typedef struct{
	int winner[THREADS];//less than 0, team A is winner.  greater than 0, team B(one for each thread)
	bool final;//true if set in the spreadsheet
} Contest;

//struct used for arguments for threads
typedef struct {
	int tid;
	int firstgame;
}ThreadParam;

void getLinePick(std::ifstream &file, int contest_num);
void loadFile();
void tallyWinners(int tid);
void genProb( int tid,int firstgame);
void printResults();

std::vector<Player*> player_list; //list of players

Contest contests[35]; 

//calculated outcomes counter for each thread
int outcomes[THREADS];
//scratch pad for calculating scores for each thread
int *scores[THREADS];
float max_outcomes;

//Thread Routine
DWORD ThreadProc (LPVOID lpdwThreadParam ) 
{ 
	ThreadParam *param = (ThreadParam*) lpdwThreadParam;

	//create thread's score array
	scores[param->tid] = new int[player_list.size()];

	//do work!
	genProb(param->tid,param->firstgame);

	//cleanup thread's score array
	delete [] scores[param->tid];
	return 0;
}


int _tmain(int argc, _TCHAR* argv[])
{
	loadFile();
	//set counters to 0
	memset(outcomes,0,sizeof(int)*THREADS);

	//find the first game
	int firstgame;
	for(firstgame = 0; firstgame < 35;firstgame++){
		if(!contests[firstgame].final)
			break;
	}
	//calculate the number of outcomes to compute
	max_outcomes = pow(2.0,(35-firstgame));

	//manually set the first game(s) differently for each thread
	//Could be improved...
	if(THREADS == 2){
		contests[firstgame].winner[0] = -1;
		contests[firstgame++].winner[1] = 1;
	}
	else if(THREADS == 4)
	{
		contests[firstgame].winner[0] = -1;
		contests[firstgame].winner[1] = -1;
		contests[firstgame].winner[2] = 1;
		contests[firstgame++].winner[3] = 1;

		contests[firstgame].winner[0] = -1;
		contests[firstgame].winner[1] = 1;
		contests[firstgame].winner[2] = -1;
		contests[firstgame++].winner[3] = 1;
	}
	else if(THREADS == 8)
	{
		contests[firstgame].winner[0] = -1;
		contests[firstgame].winner[1] = -1;
		contests[firstgame].winner[2] = -1;
		contests[firstgame].winner[3] = -1;
		contests[firstgame].winner[4] = 1;
		contests[firstgame].winner[5] = 1;
		contests[firstgame].winner[6] = 1;
		contests[firstgame++].winner[7] = 1;

		contests[firstgame].winner[0] = -1;
		contests[firstgame].winner[1] = -1;
		contests[firstgame].winner[2] = 1;
		contests[firstgame].winner[3] = 1;
		contests[firstgame].winner[4] = -1;
		contests[firstgame].winner[5] = -1;
		contests[firstgame].winner[6] = 1;
		contests[firstgame++].winner[7] = 1;

		contests[firstgame].winner[0] = -1;
		contests[firstgame].winner[1] = 1;
		contests[firstgame].winner[2] = -1;
		contests[firstgame].winner[3] = 1;
		contests[firstgame].winner[4] = -1;
		contests[firstgame].winner[5] = 1;
		contests[firstgame].winner[6] = -1;
		contests[firstgame++].winner[7] = 1;
	}
	
	ThreadParam params[THREADS];
	HANDLE th[THREADS];
	DWORD dwThreadId[THREADS];
	for(int i = 0; i < THREADS;i++)
	{
		//setup thread's parameters
		params[i].firstgame = firstgame;
		params[i].tid = i;
		
		//launch thread!
		printf("launching thread %d\n",i);
		th[i] = CreateThread(NULL, //Choose default security
			0, //Default stack size
			(LPTHREAD_START_ROUTINE)&ThreadProc,//Routine to execute
			(LPVOID) &params[i], //Thread parameter
			0, //Immediately run the thread
			&dwThreadId[i] //Thread Id
			);
	}

	
	DWORD exitCode;
	
	int total = 0;
	float ops = 0;

	//setup timing variables
	clock_t start = clock();
	float elapse,remain;

	//now wait for threads to complete
	int runningthreads = THREADS;
	while(runningthreads)
	{		
		total = 0;
		runningthreads =0;
		for(int i = 0; i < THREADS;i++){
			GetExitCodeThread(th[i],&exitCode);
			if(exitCode == STILL_ACTIVE)
			{
				runningthreads++;				
			}
			total +=outcomes[i];
		}
		elapse = (clock() - start) / CLOCKS_PER_SEC;
		remain = ((max_outcomes - total) / ((float) total / elapse))/60;
		printf("\r%f%% done (%.2f Kops) %.2fmin remain(%.2fmin elapsed)",((float)total / max_outcomes) * 100.0,(float) total / (elapse*1000),remain,elapse/60);

		Sleep(250);
	}

	printResults();
	return 0;
}

//comparison function for comparing Player structs
bool cmpPlayers (Player* left,Player* right) 
{ 
	if(left->first[0] == right->first[0])
		if(left->second[0] == right->second[0])
				if(left->third[0] == right->third[0])
					return left->fourth[0] > right->fourth[0];
				else
					return left->third[0] > right->third[0];
		else
			return left->second[0] > right->second[0];
	else
		return left->first[0] > right->first[0];	 
}

void printResults()
{
	printf("\nResults:\n");
	std::vector<Player*> order;
	for(int i = 0; i < player_list.size();i++)
	{
		for(int j = 1; j < THREADS;j++){
			player_list[i]->first[0] += player_list[i]->first[j];
			player_list[i]->second[0] += player_list[i]->second[j];
			player_list[i]->third[0] += player_list[i]->third[j];
			player_list[i]->fourth[0] += player_list[i]->fourth[j];
		}
		if( player_list[i]->first[0] || player_list[i]->second[0] || player_list[i]->third[0] || player_list[i]->fourth[0])
			order.push_back(player_list[i]);
	}
	std::sort(order.begin(),order.end(),cmpPlayers);
	
	float first,second, third,fourth;
	for(int i = 0; i < order.size();i++)
	{		
		first = ((float) order[i]->first[0] / max_outcomes)*100.0;
		second = ((float) order[i]->second[0] / max_outcomes)*100.0;
		third = ((float) order[i]->third[0] / max_outcomes)*100.0;
		fourth = ((float) order[i]->fourth[0] / max_outcomes)*100.0;
		printf("%d. 1st: %.4f\t2nd: %.4f\t3rd: %.4f\t4th: %.4f|%s\n",i+1,first,second,third,fourth,order[i]->name);
	}
}

//recursive function that determines all possible outcomes
void genProb( int tid,int firstgame)
{
	if(firstgame == 35){

		tallyWinners(tid);
		outcomes[tid]++;
		return;
	}
	else{
		contests[firstgame].winner[tid] = -1;
		genProb(tid,firstgame+1);
		contests[firstgame].winner[tid] = 1;
		genProb(tid,firstgame+1);
	}
	
}

//determines who the winners are (1st, 2nd and 3rd)
//NOTE: does not check for ties!!
void tallyWinners(int tid)
{
	memset(scores[tid],0,sizeof(int)*player_list.size());

	for(int i = 0; i < player_list.size();i++)
	{
		for(int j = 0; j < 35;j++)
			if((player_list[i]->pickedA[j] && contests[j].winner[tid] < 0) ||
			   (!player_list[i]->pickedA[j] && contests[j].winner[tid] > 0))
				scores[tid][i] += player_list[i]->rank[j];
	}
	int first,second, third,fourth;
	first=second=third=fourth=0;
	for(int i = 1;i<player_list.size();i++)
	{
		if(scores[tid][i] > scores[tid][first])
		{
			fourth = third;
			third = second;
			second = first;
			first = i;
		}
		else if(scores[tid][i] > scores[tid][second])
		{
			fourth = third;
			third = second;
			second = i;
		}
		else if(scores[tid][i] > scores[tid][third])
		{
			fourth = third;
			third = i;
		}
		else if(scores[tid][i] > scores[tid][fourth])
		{
			fourth = i;
		}
	}
	player_list[first]->first[tid]++;
	player_list[second]->second[tid]++;
	player_list[third]->third[tid]++;
	player_list[fourth]->fourth[tid]++;
	
}

void loadFile()
{
	std::ifstream file("test.csv" , std::ifstream::in);	
	file.ignore(2048,'\n');//ignore first row
	//file.ignore(2048,'\n');//ignore second row

	for(int i = 0; i < 12;i++)//ignore 12 commas
	{
		file.ignore(2048,',');
	}
	bool go = true;
	while(go)
	{
		Player *player = new Player;
		for(int i = 0; i < THREADS;i++)
		{
			player->first[i] = 0;
			player->second[i] = 0;
			player->third[i] = 0;
			player->fourth[i] = 0;
		}
		file.getline(player->name,20,',');
		file.ignore(2048,',');
		file.ignore(2048,',');
		file.ignore(2048,',');
		if(strcmp(player->name,"Brad M.") == 0)
			go = false;
		player_list.push_back(player);
	}
	file.ignore(2048,'\n');

	for(int i = 0; i < 35;i++)
		getLinePick(file,i);
}

void getLinePick(std::ifstream &file, int contest_num)
{
	bool pickedA;
	int rank;
	char temp[10];
	char teamA[10];
	for(int i = 0; i < 8;i++)//ignore 8 commas
	{
		file.ignore(2048,',');
	}
	file.getline(teamA,10,','); //get teamA

	//ignore 2 spots
	file.ignore(2048,','); file.ignore(2048,','); 
	//file.ignore(2048,','); 

	file.getline(temp,10,','); //get winner
	int size = file.gcount();
	if(size > 1)
	{
		if(strcmp(temp,teamA) == 0)
			for(int i = 0; i < THREADS;i++)
				contests[contest_num].winner[i] = -1;
		else 
			for(int i = 0; i < THREADS;i++)
				contests[contest_num].winner[i] = 1;
		
		contests[contest_num].final = true;
	}
	else
	{
		memset(contests[contest_num].winner,0,sizeof(int)*THREADS);
		contests[contest_num].final = false;
	}

		
	for(int i = 0; i < player_list.size();i++)
	{
		file.getline(temp,10,','); //get pick
		if(strcmp(temp,teamA) == 0)
			pickedA = true;
		else 
			pickedA = false;

		file.getline(temp,10,','); //get rank
		rank = atoi(temp);

		//ignore 2 spots
		if(i == player_list.size()-1)
			file.ignore(2048,'\n');
		else
		{
			file.ignore(2048,','); 
			file.ignore(2048,',');
		}
		player_list[i]->pickedA[contest_num] = pickedA;
		player_list[i]->rank[contest_num] = rank;
	}

}
