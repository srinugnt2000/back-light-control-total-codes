/**********************************************************************************************
*    Module Name 	:	Automatic Backlight ON & off
*    Description    	:  	Disable Backlight Automatically after 10 sec 
*    Function List  	:  	
*    Limitations    	:  	Rectifying
*    Workfile       	:	back-light.c
*    Soft Version   	:	Linux(2.6.32-28 Version)
*    Teted On Host 	:	Ubuntu 10.04 LTE(VMware) 
*    Date           	:   	13/09/2013
*    Author          	:   	K.Sreenivas
*
*   (C) Stesalit Ltd All rights reserved.
*-----------------------------------------------------------------------------------------------
*   Revision History
*   Date      		Who            Description
*   14-06-2013 		XYZ            First Release for Review
***********************************************************************************************/
/*---------------------------------------------------------------------------------------*/
/* INCLUDES */
/*---------------------------------------------------------------------------------------*/
#include <stdio.h>
#include <stdlib.h>
#include <linux/input.h>
#include <signal.h>
#include <sched.h>
#include <string.h>
#include <linux/sched.h>


#define STACK_SIZE 16000

static volatile sig_atomic_t gotAlarm = 0;	
static volatile sig_atomic_t lcd_off_flag=1;
struct itimerval itv;
static int th_flag=0;
static int kb_flag=0;
unsigned long int min,min1,min2;
static int bug=1;


/********************Signal Handler for  Child *********************/
static void sigalthHandler(int sig)
{
	th_flag=1;
//    printf(" got sigalarm for key board %d times \n", ++gotAlarm);
}
/********************Signal Handler for  Child**********************/


/********************Signal Handler for Parent  *********************/
static void sigalkbHandler(int sig)
{
	kb_flag=1;
//    printf(" got sigalarm for key board %d times \n", ++gotAlarm);
}
/********************Signal Handler for Parent **********************/




/*******************Child Process for Touch Screen handling*********************/
int chld_process(void *arg)
{	
	struct input_event evth;

	FILE *kth = fopen("/dev/input/event1", "r");		//touch screen event
//	printf("\nIn child process\n");


	struct sigaction sath;
    sigemptyset(&sath.sa_mask);
    sath.sa_flags = SA_SIGINFO;
    sath.sa_handler = sigalthHandler;
    if (sigaction(SIGALRM, &sath, NULL) == -1)
        perror("sigaction");

	
	if (setitimer(ITIMER_REAL, &itv, NULL) == -1){
       		perror("setitimer : ");
			exit(1);
			}  

  while(1)
	{
	if (th_flag==1 && kb_flag==1)
	lcd_off_flag=0;
	
	if (lcd_off_flag==0)
	{
	if (bug == 1)
	{
	bug = 0;
	system("start-script");
	}
	system("i2cset -f -y 1 0x48 0x18 0x00");
	system("i2cset -f -y 1 0x48 0x19 0x00");
	}		
	while ((fread(&evth, sizeof(evth), 1, kth) == 1))
	{	
		th_flag=0;
		if (lcd_off_flag == 1)
		{
		if (setitimer(ITIMER_REAL, &itv, NULL) == -1){
       		perror("setitimer : ");
			exit(1);
			}  	
//    		printf("ev.code:%d",evth.code); 	
//    		printf("F1 Key has pressed\n");
		}
	}
    }

  fclose(kth);
}
/*******************Child Process for Touch Screen handling*********************/
//#define SIZE 2
//#define NUMELEM 3


/*******************Parent Process for Key board handling***********************/
int main (int argc, char *argv[])
{

FILE* fd = NULL;
//char buff[4];	
int size=0;
char *buff;

void *child_stack;


	/*get memory for child stack*/
	child_stack= malloc(STACK_SIZE);
	if(child_stack==NULL){
		perror("\nError creating child stack\n");
		exit(1);
		}
/*******************************read the number of minuites**********/
		
		fd = fopen("/usr/lib/test.txt","r");
		if(NULL == fd)
		    {
	        printf("\n fopen() Error!!!\n");
	        min = 60;
		goto jump;
		    }
		/***************read-length-of-file*************/
				
		fseek(fd, 0, SEEK_END); // seek to end of file
		size = ftell(fd); // get current file pointer
		printf("the size if %d",size);
		fseek(fd, 0, SEEK_SET); // seek back to beginning of file
		buff=(char*)malloc((size)*sizeof(char)); 
		if(buff==NULL) { printf("Error! memory not allocated."); exit(0); } 
		memset(buff,0,(size));
		/***************read-length-of-file*************/

		printf("\n File opened successfully through fopen()\n");
		
		if(sizeof(char)*size != fread(buff,sizeof(char),size,fd))  
  		{
        	printf("\n fread() failed\n");
	        min = 60;
		goto jump;
   		}
		fclose(fd);
		printf("\n fread() sucessfuly done");
		buff[size] = '\0';
		printf("\n the read string is%sthe",buff);
		printf("\n the size of buffer is %d", sizeof(buff));

/*
		min = buff[0];
		min = (min - 48)*100;
		min1 = buff[1];
		min1 = (min1 - 48)*10;
		min2 = buff[2];
		min2 = (min2 - 48);
		min = (min + min1 + min2);
*/

		min = strtoul(buff, NULL, 0);		

jump:		printf("\n the number of min read is %lu \n", min);

/*******************************read the number of minuites**********/
		
  		itv.it_value.tv_sec = min;  // initial event
		itv.it_value.tv_usec = 0;
		itv.it_interval.tv_sec = min;// periodic interval 
		itv.it_interval.tv_usec = 0;
  		if (setitimer(ITIMER_REAL, &itv, NULL) == -1){
       		perror("setitimer : ");
			exit(1);}  
	bug = 1;	

/****************************creating child process*****************************/
 clone( chld_process, child_stack+STACK_SIZE, CLONE_VM|CLONE_FILES, NULL);
/****************************creating child process ****************************/	

struct sigaction sakb;
    sigemptyset(&sakb.sa_mask);
    sakb.sa_flags = SA_SIGINFO;
    sakb.sa_handler = sigalkbHandler;
    if (sigaction(SIGALRM, &sakb, NULL) == -1)
        perror("sigaction");

  struct input_event evbd;
  FILE *kbd = fopen("/dev/input/event0", "r");		//key-board-event
  
  while (1)
    {	
	if (th_flag==1 && kb_flag==1)
	lcd_off_flag=0;
	
	if (lcd_off_flag==0)
	{
	if (bug == 1)
	{
	system("start-script"); 
	bug = 0;
	}
	system("i2cset -f -y 1 0x48 0x18 0x00");
	system("i2cset -f -y 1 0x48 0x19 0x00");
	}	
				
	while ((fread(&evbd, sizeof(evbd), 1, kbd) == 1)) 
	{
	kb_flag=0;
	if (lcd_off_flag == 1)
		{
		if (setitimer(ITIMER_REAL, &itv, NULL) == -1){
       		perror("setitimer : ");
			exit(1);}  	
//		printf("ev.code:%d",evbd.code);
//		printf("F1 Key has pressed\n");
		}
	
	if (evbd.type == EV_KEY  && lcd_off_flag == 0 )
		{
			lcd_off_flag = 1;
			bug = 1;
			system("stop-script");
			system("i2cset -f -y 1 0x48 0x18 0x90");
			system("i2cset -f -y 1 0x48 0x19 0x32");
			kb_flag=0;
			th_flag=0;
		}
	}			
   }
  fclose(kbd);
  return 0;
}
/*******************Parent Process for Key board handling***********************/

