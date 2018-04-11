#include<iostream>
#include<algorithm>
using namespace std;
struct gant{
	int response;
	int pro;
	int comp;
	gant *next;
	gant *prev;
};
gant *start=NULL;
gant *end=NULL;

struct node{
	int data;
	node *link;
};
node *front=NULL;
node *rear=NULL;

int n,tq,less_tq = 0,idle = 0,pos = 0,complete=0;
int* arrival;
int** ctw;
int* burst;
int* process;
int* priority;
int** propri;
int** proburst;

void insert_node(int** sort_arrive,int l)
{
	int addp,addb;
	gant *p1=new gant;
	if(start == NULL)
	{
		p1->response = 0;
		addp = sort_arrive[l][0];
		p1->pro = addp;
		
		if(sort_arrive[l][3] >= tq)
		{
			p1->comp = tq;
		}
		else
		{
			addb = sort_arrive[l][3];
			p1->comp =addb;
		}
		
		p1->next =NULL;
		p1->prev = NULL;
		if(start == NULL)
		{
			start=end=p1;
		}
	}
	else
	{
		p1->response = end->comp;
		addp = sort_arrive[l][0];
		p1->pro = addp;
		
		if(sort_arrive[l][3] >= tq)
		{
			p1->comp = end->comp + tq;
		}
		else
		{
			addb = sort_arrive[l][3];
			p1->comp = end->comp + addb;
		}
	
		p1->next = NULL;
		p1->prev = NULL;
		end->next = p1;
		p1->prev = end;
		end = p1;
	}	
}


void add_node(int l)
{
	int addp,addb;
	gant *p1=new gant;
	if(start == NULL)
	{
		p1->response = 0;
		addp = process[l];
		p1->pro = addp;
		
		if(less_tq == 0)
		{
			p1->comp = tq;
		}
		else
		{
			addb = burst[l];
			p1->comp =addb;
		}
		
		p1->next =NULL;
		p1->prev = NULL;
		if(start == NULL)
		{
			start=end=p1;
		}
	}
	else
	{
		p1->response = end->comp;
		addp = process[l];
		p1->pro = addp;
		
		if(proburst[l][1] >= tq)
		{
			p1->comp = end->comp + tq;
		}
		else
		{
			addb = proburst[l][1];
			p1->comp = end->comp + addb;
		}
	
		p1->next = NULL;
		p1->prev = NULL;
		end->next = p1;
		p1->prev = end;
		end = p1;
	}	
}

void enqueue(int ele)
{
	node *p=new node;
	p->data	= ele;
	p->link=NULL;
	if(front == NULL)
	{
		front=rear=p;
	}
	else
	{
		rear->link=p;
		rear=p;
	}
}

int dequeue()
{
	int item;
	node *old;
	
	if(front == NULL)
	{
		cout<<"underflow"<<endl;
	}
	else
	{
		old=front;
		item=old->data;
		front=front->link;
		delete(old);
	}
	
	return item;	
}

void similar()
{
	
	int spro,spri,pro1,loc,last_loc;
	
	propri=new int*[n];
	for(int i=0;i<n;i++)
	{
		propri[i]=new int[2];
	}
	proburst=new int*[n];
	for(int i=0;i<n;i++)
	{
		proburst[i]=new int[2];
	}
	
	for(int i=0;i<n;i++)
	{
		propri[i][0] = process[i];
		proburst[i][0] = process[i];
		proburst[i][1] = burst[i];
		propri[i][1] = priority[i];
	}

	for(int i=0;i<n;i++)
	{
		for(int j=i+1;j<n;j++)
		{
			if(propri[i][1] < propri[j][1])
			{
				
				spro=propri[i][0];
				propri[i][0] = propri[j][0];
				propri[j][0] = spro;
				
				spri=propri[i][1];
				propri[i][1] = propri[j][1];
				propri[j][1] = spri;
			}
		}
	}
	for(int i=0;i<n;i++)
	{
		enqueue(propri[i][0]);
	}
	
	node *q=front;
	while(q != NULL)
	{
		
		pro1 = dequeue();
		
		for(int i=0;i<n;i++)
		{
			if(pro1 == proburst[i][0])
			{
				loc=i;
			}
		}
		
		if(proburst[loc][1] < tq)
		{
			less_tq = 1;
			add_node(loc);
			proburst[loc][1]=proburst[loc][1]-proburst[loc][1];
		}
		else
		{
			add_node(loc);
			proburst[loc][1]=proburst[loc][1]-tq;
			
			if(proburst[loc][1] != 0)
			{
			
				
				enqueue(proburst[loc][0]);
			
				last_loc=loc;
			}
		}
		q=front;
		q=q->link;
	}

	gant *p2=new gant;
	p2->pro = proburst[last_loc][0];
	p2->response = end->comp;
	p2->comp = proburst[last_loc][1] + end->comp;
	p2->next = NULL;
	p2->prev = NULL;
	end->next=p2;
	p2->prev=end;
	end=p2;
	

	cout<<"\n\n\t-------------------------------------------------------------Gant Chart ----------------------------------------------------------------------"<<endl;
	gant *dis = start;
	if(idle == 0)
	{
		cout<<"\n\n\t"<<dis->response<<"--->"<<"p"<<dis->pro<<"--->"<<dis->comp<<"--->";
	}
	else
	{
		cout<<"\n\n\t"<<dis->response<<"--->"<<"Idle"<<"--->"<<dis->comp<<"--->";
	}
	dis = dis->next;
	while(dis != NULL)
	{
		cout<<"p"<<dis->pro<<"--->"<<dis->comp<<"--->";
		dis = dis->next;
	}
	cout<<"\n\n\t--------------------------------------------------------------------------------------------------------------------------------------------"<<endl;
}


void add_idle(int** arr,int l1)
{
	cout<<"inside idle function"<<endl;
	int same_arrive = 0,spro,spri;
	if(start == NULL)
	{
		gant *p1 = new gant;
		p1->response = 0;
		p1->pro = 404;
		p1->comp = arr[l1][1];
		p1->next =NULL;
		p1->prev =NULL;
		if(start == NULL)
		{
			start=end=p1;
		}
		cout<<"idle node is added to gant"<<endl;
	}
	else
	{
		same_arrive=0;
		for(int i=0;i<n;i++)
		{
			if( (arr[i][1] <= end->comp) && (arr[i][3] > 0) )
			{
				same_arrive++;
			}	
		}	
		if(same_arrive >= 1)
		{
			int** sort_prio1 = new int*[same_arrive];
			int ind1=0;
			for(int j=0;j<same_arrive;j++)
			{
				if( (arr[j][1] <= end->comp) && (arr[j][3] > 0) )
				{
					sort_prio1[ind1][0] = arr[j][0];
					sort_prio1[ind1][1] = arr[j][2];
					ind1++;
				}
			}
			for(int i=0;i<same_arrive;i++)
			{
				for(int j=i+1;j<same_arrive;j++)
				{
					if(sort_prio1[i][1] < sort_prio1[j][1])
					{
				
						spro=sort_prio1[i][0];
						sort_prio1[i][0] = sort_prio1[j][0];
						sort_prio1[j][0] = spro;
				
						spri=sort_prio1[i][1];
						sort_prio1[i][1] = sort_prio1[j][1];
						sort_prio1[j][1] = spri;
					}
				}
			}
			int take =0;
			while(end->comp >= arr[l1][1])
			{
				gant *p1 = new gant;
				p1->response = end->comp;
				p1->pro = sort_prio1[take][0];
				if(sort_prio1[take][2] >= tq)
				{
					p1->comp = end->comp + tq;		
				}
				else
				{
					p1->comp = end->comp + arr[take][2];	
				}
			
			p1->next =NULL;
			p1->prev =NULL;
			end->next = p1;
			p1->prev = end;
			end = p1;
				take++;
			}
			
		}
		else
		{
			gant *p1 = new gant;
			p1->response = end->comp;
			p1->pro = 404;
			p1->comp = arr[l1][1];
			p1->next =NULL;
			p1->prev =NULL;
			end->next = p1;
			p1->prev = end;
			end = p1;
		}
		cout<<"idle node is added to gant"<<endl;
	}
	
	
}


void add_all(int** a)
{
	cout<<"inside add all "<<endl;
	int num = 0,index = 0,s1,s2,s3,element,pos1,again=0,change=0;
	for(int i=0;i<n;i++)
	{
		if(a[i][3] > 0)
		{
			num++;
		}
	}
	cout<<"non zero values :"<<num<<endl;
	int** b=new int*[num];
	for(int i=0;i<num;i++)
	{
		b[i]=new int[3];
	}
	for(int i=0;i<n;i++)
	{
		if(a[i][3] > 0)
		{
			b[index][0]=a[i][0];
			b[index][1]=a[i][2];
			b[index][2]=a[i][3];
			index++;
		}
		
	}
	cout<<"non zero burst time array :"<<endl;
	for(int i=0;i<num;i++)
	{
		cout<<b[i][0]<<"\t"<<b[i][1]<<"\t"<<b[i][2]<<"\t"<<endl;
	}
	for(int i=0;i<num;i++)
	{
		for(int j=i+1;j<num;j++)
		{
			if(b[i][1] < b[j][1])
			{
				s1=b[i][0];
				b[i][0] = b[j][0];
				b[j][0]=s1;
				
				s2=b[i][1];
				b[i][1] = b[j][1];
				b[j][1]=s2;
				
				s3=b[i][2];
				b[i][2] = b[j][2];
				b[j][2]=s3;
			}
		}
	}
	cout<<"b aaray is "<<endl;
	for(int i=0;i<num;i++)
	{
		cout<<b[i][0]<<"\t"<<b[i][1]<<"\t"<<b[i][2]<<endl;
	}
	for(int i=0;i<num;i++)
	{
		enqueue(b[i][0]);
	}
	cout<<"all enqueued"<<endl;
	 node *show = front;
	 int plus =0;
	 while(show != NULL)
	 {
	 	if(plus == num)
	 	{
	 		plus = 0;
		 }
		element = dequeue();
		cout<<"element dequeued is :"<<element<<endl;
	
		for(int i=0;i<num;i++)
		{
			if(b[i][2] > 0)
			{
				plus = i;
				break;
			}
		}
		cout<<"count is :"<<plus<<endl;
				gant *ad =new gant;
		ad->response = end->comp;
		ad->pro = element;
		ad->next =NULL;
		ad->prev = NULL;
		if(b[plus][2] >= tq)
		{
			ad->comp = end->comp + tq;
			end->next = ad;
			ad->prev =end;
			end= ad;
	 		b[plus][2] =b[plus][2]-tq;
	 		cout<<"burst time after subtracting tq : "<<b[plus][2];
			 if(b[plus][2] != 0)
			{
				enqueue(element);
			} 
		}
		else
		{
			ad->comp = end->comp + b[plus][2];
			end->next = ad;
			ad->prev =end;
			end= ad;
		 	b[plus][2]=b[plus][2]-b[plus][2];
		 	cout<<"burst time after subtracting : "<<b[plus][2];
		 	if(b[plus][2] != 0)
			{
				cout<<"enqueueing"<<b[plus][2]<<endl;
				enqueue(element);
			}
		}
		
		
	//	plus++;
		show = show->link;
	 }
	 
	cout<<"all added into gant from add all function"<<endl;
	for(int i=0;i<num;i++)
	{
		if(b[i][2] > 0)
		{
		again = i;
		change=1;
		}
	}
	
	if(change == 1){
		
	gant *ad =new gant;
			ad->response = end->comp;
			ad->pro = b[again][0];
			ad->next=NULL;
			ad->prev=NULL;
			if(b[again][2] >= tq)
			{
				ad->comp = end->comp + tq;
				end->next = ad;
				ad->prev =end;
				end= ad;
	 			b[again][2] =b[again][2]-tq;
	 				 	
			}
			else
			{
				ad->comp = end->comp + b[again][2];
				end->next = ad;
				ad->prev =end;
				end= ad;
		 		b[again][2]=b[again][2]-b[again][2];
		 		
			}
		}
		
	//cout<<"element is queue still presetn :"<<show->data<<endl;
}

void different()
{
	cout<<"inside different"<<endl;
	int gant_value,spro1,spri1,arri,cpu,spro,sprinsame_arrive=0,pr,f_match = 0,first_match,change_prio,change_pro,same_prio=0,spri,same_arrive;
	
	int *pri=new int[n];
	for(int i=0;i<n;i++)
	{
		pri[i]=priority[i];
	}
	
	sort(pri,pri+n,greater<int>());
	
	int** proarrive = new int*[n];
	for(int i=0;i<n;i++)
	{
		proarrive[i]=new int[4];
	}
	for(int i=0;i<n;i++)
	{
		proarrive[i][0] = process[i];
		proarrive[i][1] = arrival[i];
		proarrive[i][2] = priority[i];
		proarrive[i][3] = burst[i];
	}
	
	for(int i=0;i<n;i++)
	{
		for(int j=i+1;j<n;j++)
		{
			if(proarrive[i][1] > proarrive[j][1])
			{
				
				spro1=proarrive[i][0];
				proarrive[i][0] = proarrive[j][0];
				proarrive[j][0] = spro1;
				
				arri=proarrive[i][1];
				proarrive[i][1] = proarrive[j][1];
				proarrive[j][1] = arri;
				
				spri1=proarrive[i][2];
				proarrive[i][2] = proarrive[j][2];
				proarrive[j][2] = spri1;
				
				cpu=proarrive[i][3];
				proarrive[i][3] = proarrive[j][3];
				proarrive[j][3] = cpu;
			}
		}
	}
	cout<<"\n\n\tsorted proarrive :"<<endl;
	cout<<"\n\tprocess\tarrival\tpriority\tburst"<<endl;
	for(int i=0;i<n;i++)
	{
		cout<<"\t"<<proarrive[i][0]<<"\t"<<proarrive[i][1]<<"\t"<<proarrive[i][2]<<"\t"<<proarrive[i][3]<<endl;
	}
	for(int i=0;complete!=n;i++)
	{
		cout<<"inside main for loop"<<endl;
		if(start == NULL)
		{
			gant_value = 0;
		}
		else
		{
			gant_value = end->comp;
		}
		if(gant_value >= proarrive[n-1][1])
		{
			cout<<"inside add_all"<<endl;
			add_all(proarrive);
			break;
		}
		if(i == n-1)
		{
			i=0;
		}
		pos = i;
		cout<<"gant_value is :"<<gant_value<<endl;
		if( (proarrive[pos][1] <= gant_value) && (proarrive[pos][3] > 0) )
		{
			
			cout<<"inside if in main "<<endl;
			same_arrive = 0;
			
			for(int j=0;j<n;j++)
			{
				if( (proarrive[j][1] <= gant_value) && (proarrive[j][3] > 0) )
				{
					same_arrive++;
					//same_arrive = j;
				}
			}
			cout<<"same values (same_arrive)are :"<<same_arrive<<endl;
			int** sort_prio = new int*[same_arrive];
			for(int k=0;k<same_arrive;k++)
			{
				sort_prio[k]=new int[2];
			}
			int ind=0;
			for(int j=0;j<n;j++)
			{
				if( (proarrive[j][1] <= gant_value) && (proarrive[j][3] > 0) )
				{
					sort_prio[ind][0] = proarrive[j][0];
					sort_prio[ind][1] = proarrive[j][2];
					ind++;
				}
			}
			cout<<"sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio[j][0]<<"\t"<<sort_prio[j][1]<<endl;
			}
			for(int i=0;i<same_arrive;i++)
			{
				for(int j=i+1;j<same_arrive;j++)
				{
					if(sort_prio[i][1] < sort_prio[j][1])
					{
				
						spro=sort_prio[i][0];
						sort_prio[i][0] = sort_prio[j][0];
						sort_prio[j][0] = spro;
				
						spri=sort_prio[i][1];
						sort_prio[i][1] = sort_prio[j][1];
						sort_prio[j][1] = spri;
					}
				}
			}
			cout<<"sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio[j][0]<<"\t"<<sort_prio[j][1]<<endl;
			}
			for(int j=0;j<same_arrive-1;j++)
			{
				if(sort_prio[j][1] == sort_prio[j+1][1] )
				{
					if(f_match == 0){
						first_match = j;
						f_match = 1;	
					}
					
					same_prio = j+1;
					//change = 1;
				}
			}
			cout<<"first match is :"<<first_match<<"same_prio is :"<<same_prio<<endl;
			int k=0;
			for(int j=same_prio;j>first_match;j--)
			{
								
				change_pro = sort_prio[k][0];
				change_prio = sort_prio[k][1];
					
				sort_prio[k][0] = sort_prio[j][0];
				sort_prio[k][1] = sort_prio[j][1];
					
				sort_prio[j][0] = change_pro;
				sort_prio[j][1] = change_prio;
			
				k++;
				
			}
			cout<<"sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio[j][0]<<"\t"<<sort_prio[j][1]<<endl;
			}
			for(int j=0;j<same_arrive;j++)
			{
				pos = sort_prio[j][0];
				enqueue(pos);
			}
			
			cout<<"all node enqueued "<<endl;
			for(int k=0;k<same_arrive;k++)
			{
				
			
				pr = dequeue();
				cout<<"dequeued node is :"<<pr<<endl;
				for(int j=0;j<n;j++)
				{
					if(pr == proarrive[j][0])
					{
						pos = j;
					}
				}
				cout<<"pos is :"<<pos<<endl;
				insert_node(proarrive,pos);
				if(proarrive[pos][3] >= tq)
				{
					cout<<"subtracting"<<endl;
					proarrive[pos][3] = proarrive[pos][3]-tq;
					if(proarrive[pos][3] == 0)
					{
						complete++;
					}
				}
				else
				{
					cout<<"subtracting"<<endl;
					proarrive[pos][3] = proarrive[pos][3]-proarrive[pos][3];
					if(proarrive[pos][3] == 0)
					{
						complete++;
					}
					
				}
				
				
			}
		
		}
		else
		{
			
			add_idle(proarrive,pos);
			cout<<"inside main in idle part"<<endl;
			for(int j=0;j<n;j++)
			{
				if( (proarrive[j][1] <= end->comp) && (proarrive[j][3] > 0) )
				{
					same_arrive++;
				}
			}
			cout<<"same arrive : "<<same_arrive<<endl;
			int** sort_prio1 = new int*[same_arrive];
			for(int k=0;k<same_arrive;k++)
			{
				sort_prio1[k]=new int[2];
			}
			int ind1=0;
			for(int j=0;j<n;j++)
			{
			//	cout<<"loop setting the value of sort_arrive"<<endl;
				if( (proarrive[j][1] <= end->comp) && (proarrive[j][3] > 0) )
				{
					sort_prio1[ind1][0] = proarrive[j][0];
				
					//cout<<"sort_prio"<<sort_prio1[ind1][0]<<endl;
					sort_prio1[ind1][1] = proarrive[j][2];
					ind1++;
				}
			}
			cout<<"sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio1[j][0]<<"\t"<<sort_prio1[j][1]<<endl;
			}
			for(int i=0;i<same_arrive;i++)
			{
				for(int j=i+1;j<same_arrive;j++)
				{
					if(sort_prio1[i][1] < sort_prio1[j][1])
					{
				
						spro=sort_prio1[i][0];
						sort_prio1[i][0] = sort_prio1[j][0];
						sort_prio1[j][0] = spro;
				
						spri=sort_prio1[i][1];
						sort_prio1[i][1] = sort_prio1[j][1];
						sort_prio1[j][1] = spri;
					}
				}
			}
			cout<<"sorted sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio1[j][0]<<"\t"<<sort_prio1[j][1]<<endl;
			}
			cout<<"same_prio :"<<same_prio<<endl;
			for(int j=0;j<same_arrive-1;j++)
			{
				cout<<"inside check loop for equal priority "<<endl;
				if(sort_prio1[j][1] == sort_prio1[j+1][1] )
				{
					if(f_match == 0){
						first_match = j;
						f_match = 1;	
					}
					
					same_prio = j+1;
					//change = 1;
				}
			}
			
			int l=0;
			cout<<"value of first match"<<first_match<<"\n same prio is :"<<same_prio<<endl;
			for(int j=same_prio;j>first_match;j--)
			{
							cout<<"inside first match check loop"<<endl;	
				change_pro = sort_prio1[l][0];
				change_prio = sort_prio1[l][1];
					
				sort_prio1[l][0] = sort_prio1[j][0];
				sort_prio1[l][1] = sort_prio1[j][1];
					
				sort_prio1[j][0] = change_pro;
				sort_prio1[j][1] = change_prio;
			
				l++;
				
			}
			cout<<"descending sort prio array is :"<<endl;
			for(int j=0;j<same_arrive;j++)
			{
				cout<<sort_prio1[j][0]<<"\t"<<sort_prio1[j][1]<<endl;
			}
			for(int j=0;j<same_arrive;j++)
			{
				pos = sort_prio1[j][0];
				enqueue(pos);
			}
			cout<<"all node enqueued"<<endl;
			
			for(int k=0;k<same_arrive;k++)
			{
				
			
				pr = dequeue();
				cout<<"dequeued node is : "<<pr<<endl;
				for(int j=0;j<n;j++)
				{
					if(pr == proarrive[j][0])
					{
						pos = j;
					}
				}
				cout<<"pos is :"<<pos<<endl;
				insert_node(proarrive,pos);
				cout<<"node is inserted"<<endl;
				if(proarrive[pos][3] >= tq)
				{
					cout<<"subtracting tq from burst time"<<endl;
					proarrive[pos][3] = proarrive[pos][3]-tq;
					if(proarrive[pos][3] == 0)
					{
						complete++;
					}
				}
				else
				{
					cout<<"subtracting  from burst time"<<endl;
					proarrive[pos][3] = proarrive[pos][3]-proarrive[pos][3];
					if(proarrive[pos][3] == 0)
					{
						complete++;
					}
				}
				
			
		}
	/*	if(proarrive[pos][3] == 0)
		{
			complete++;
		}
		*/
	}
}
}
int main()
{
	int count;
	
	cout<<"enter no. of process :";
	cin>>n;
	cout<<"enter Time Quantum :";
	cin>>tq;
	
	arrival=new int[n];
	burst=new int[n];
	process=new int[n];
	priority=new int[n];
ctw = new int*[n];
	for(int i=0;i<n;i++)
	{
		ctw[i]= new int[4];
	}
	for(int i=0;i<n;i++)
	{
		process[i]=i+1;
		ctw[i][0]=process[i];
		cout<<"enter Burst time of process "<<i+1<<" : ";
		cin>>burst[i];
		cout<<"enter Arrival time of process "<<i+1<<" : ";
		cin>>arrival[i];
		cout<<"enter Priority of process "<<i+1<<" : ";
		cin>>priority[i];
	}
	cout<<"\n\n\t\tProcess\tArrival Time\tBurst Time\tPriority"<<endl;
	for(int i=0;i<n;i++)
	{
		cout<<"\t\t"<<process[i]<<"\t\t"<<arrival[i]<<"\t\t"<<burst[i]<<"\t"<<priority[i]<<endl;
	}
	
	for(int i=0;i<n;i++)
	{
		if(arrival[i] == arrival[0])
		{
			count++;
		}
	}
	
	if(count == n)
	{
		if(arrival[0] == 0)
		{
			similar();	
		}
		else
		{
			
			idle = 1;
			gant *p1 = new gant;
			p1->response = 0;
			p1->comp = arrival[0];
			p1->next =NULL;
			p1->prev =NULL;
			if(start == NULL)
			{
				start=end=p1;
			}
			similar();
		}
		

		
	}
	else
	{
		different();
		cout<<"\n\n\t-------------------------------------------------------------Gant Chart ----------------------------------------------------------------------"<<endl;
		gant *dis = start;
	
		cout<<"\t";
			cout<<dis->response<<"--->"<<"p"<<dis->pro<<"--->"<<dis->comp<<"--->";
		while(dis != NULL)
		{
			cout<<"p"<<dis->pro<<"--->"<<dis->comp<<"--->";
			
			//cout<<"p"<<dis->pro<<"--->"<<dis->comp<<"--->";
			dis = dis->next;
		}
		cout<<"\n\n\t--------------------------------------------------------------------------------------------------------------------------------------------"<<endl;
	
	
	cout<<"\n\n\tprocess\tcompilation Time\tturn around time\twaiting time"<<endl;
	
	//node *last_occur,*occur;
	
	//last_occur = start;
	
	
	
	}
	for(int i=0;i<n;i++)
	{
		gant *last_occur,*occur;
		occur = start;
		while(occur != NULL)
		{
			if(occur->pro == ctw[i][0])
			{
				last_occur = occur;
			}
			occur = occur->next;
		}
		
		ctw[i][1] = last_occur->comp;
		ctw[i][2] = ctw[i][1] - arrival[i];
		ctw[i][3] = ctw[i][2] - burst[i];
	}
	
	for(int i=0;i<n;i++)
	{
		cout<<"\t"<<ctw[i][0]<<"\t\t"<<ctw[i][1]<<"\t\t\t"<<ctw[i][2]<<"\t\t"<<ctw[i][3]<<endl;
	}
	
	int sum1=0,sum2=0;
	for(int i=0;i<n;i++)
	{
		sum1 = sum1 + ctw[i][3];
		sum2 = sum2 + ctw[i][2];
	}
	cout<<"\n\n\tAverage Waiting Time is :"<<(sum1 / n)<<endl;
	cout<<"\n\tAverage Turn Around Time is :"<<(sum2 / n)<<endl;
	
}
