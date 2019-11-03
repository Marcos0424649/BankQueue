//Phase II Testing Program
#include <iostream>
#include <iomanip>
#include"DataElement.h"
#include"SortedList.h"
#include"Event.h"
#include"Queue.h"


using namespace std;

const int SIZE = 10;//CONSTANT USED FOR ARRAYS

void processArrival(int& customer, DataElement  iFile[], int  length, SortedList<Event>& eList, Queue<DataElement>& bQueue);
void processDeparture(SortedList<Event>& eList, Queue<DataElement>& bQueue, int& wTime);

int main()
{
	//Array for the Arrival time and transaction length
	DataElement bankData[SIZE] = { DataElement(1,7), DataElement(2,5), DataElement(4,3), DataElement(20,5), DataElement(31,6), 
		DataElement(37,2), DataElement(41,5), DataElement(43,7), DataElement(45,3), DataElement(53,2) };


	Queue<DataElement> bankQueue;//Declares an instance of class template Queue
	SortedList<Event> eventList;//Declares an instance of class template SortedList

	int customer = 0;
	int transactionTime[SIZE];
	int waitingTime;
	int counter = 1;
	int departureTimes[SIZE];
	int waitTimes[SIZE];
	double totalWaitingTime = 0;
	int transactionStart[SIZE];

	//Create an arrival event for customer[0]
	Event aEvent1('A', bankData[customer].getArrivalTime());
	eventList.insertSorted(aEvent1);//Insert arrival event into the eventList
	waitTimes[0] = 0;/* Customer[0] walked into an empty bank, their wait time is ZERO, so add their wait time to
						the waitTime array before execution of the event while loop */

						//Event while loop
	while (!eventList.isEmpty())
	{
		Event eStatus = eventList.getEntry(1);/* Stores entry 1 from eventList into eStatus to check the status in the following
												 if else statement*/

		if (eStatus.getEventStatus() == 'A')
		{
			processArrival(customer, bankData, SIZE, eventList, bankQueue);// Function Call
			customer++;
		}
		else
		{
			if (counter != SIZE + 1)//added
			{
				// counter minus explained below 
				departureTimes[counter - 1] = eventList.getEntry(1).getOccurTime();
			}

			processDeparture(eventList, bankQueue, waitingTime);

			if (counter != SIZE)//added
			{
				waitTimes[counter] = waitingTime;// Waiting time array 
				totalWaitingTime += waitingTime;// Total waiting time to be used for average wait time
				counter++;/* used for accessing all six elements within the DepartureTimes array and the minus 1 is used becuase that array had
							 start 1 unit behind the original 1 that the counter was initilaized to before the start of the event while loop*/
			}
		}

	}
	for (int k = 0; k < SIZE; k++)
	{
		transactionStart[k] = bankData[k].getArrivalTime() + waitTimes[k];
	}


	//**************************************************************************************************************************************
	//*******************************************display the information********************************************************************
	cout << "The computer simualtion will commence....." << endl;
	cout << "The data is summerized as follows:" << endl;
	cout << "================================================================================================" << endl;
	cout << setw(2) << "Customer";
	cout << setw(20) << "Arrival Time" << setw(26) << "Transaction Begins" << setw(22) << "Departure Time" << setw(20) << "Waiting Time" << endl;
	for (int i = 0; i < SIZE; i++)
	{
		cout << setw(2) << i + 1 << setw(17) << bankData[i].getArrivalTime() << setw(20) << transactionStart[i] << setw(26) << departureTimes[i] << setw(21);
		cout << waitTimes[i] << endl;
	}
	cout <<  "------------------------------------------------------------------------------------------------" <<  endl;
	cout << "The average wait time of the 6 customers is " << totalWaitingTime / SIZE << endl;
	cout << "Simulaiton is complete" << endl;

	//system("pause");
	return 0;
}
void processArrival(int& customer, DataElement iFile[], int length, SortedList<Event>& eList, Queue<DataElement>& bQueue)
{
	if (bQueue.isEmpty())
	{
		// Enters this IF statment only if the  BANK LINE IS EMPTY 
		Event eDeparture('D', iFile[customer].getArrivalTime() + iFile[customer].getTransactionTime()); //Create and set the departure event
		eList.insertSorted(eDeparture); // Inserts the departure event into the event list
	}

	bQueue.enqueue(iFile[customer]);// Inserts a customer into the bQueue
	eList.remove(1);
	if (customer != length - 1) // this makes sure there are still items in the bank data array
	{
		Event aEvent('A', iFile[customer + 1].getArrivalTime());// create the newest arrival event
		eList.insertSorted(aEvent);// inserts the newest arrival event in a sorted order
	}
}

void processDeparture(SortedList<Event>& eList, Queue<DataElement>& bQueue, int& wTime)
{
	bQueue.dequeue();// Keep here
	if (!bQueue.isEmpty())
	{
		Event dEvent('D', eList.getEntry(1).getOccurTime() + bQueue.peekFront().getTransactionTime());// Creates a departure event and sets it
		eList.insertSorted(dEvent);// Inserts the departure event into the event list

		wTime = eList.getEntry(1).getOccurTime() - bQueue.peekFront().getArrivalTime();
	}
	else
	{
		wTime = 0;
	}
	eList.remove(1);
}
