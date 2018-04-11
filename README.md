#include <stdlib.h>
#include <stdio.h>
#include <pthread.h>

typedef enum {
LEFT=0,
RIGHT=1,
STRAIGHT=2
} turn_direction_t;

typedef enum {
NORTH=0,
SOUTH=1,
EAST=2,
WEST=3
} orientation_t;

static void gostraight(orientation_t, unsigned int);
static void turnleft(orientation_t, unsigned int);
static void turnright(orientation_t, unsigned int);
static void * approachintersection(void*);

pthread_mutex_t NW_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t NE_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t SE_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t SW_mutex = PTHREAD_MUTEX_INITIALIZER;


void lock(CAR, MUTEX) 
{do 
{         
if (pthread_mutex_lock(&MUTEX)) { 
  fprintf(stderr, "CAR LOCKED %d\n\n", CAR);     
} 
} while(0);}

void unlock(CAR, MUTEX) 
{do 
{     
 if (pthread_mutex_unlock(&MUTEX)) { 
  fprintf(stderr, "CAR UNLOCKED %d\n\n", CAR);  
} 
} while(0);}

static void gostraight(orientation_t cardirection, unsigned int carnumber)
{
switch(cardirection){
case NORTH:
lock(carnumber,NE_mutex);
lock(carnumber,SE_mutex);
printf("Car %d, Moving South-North\n\n", carnumber);
unlock(carnumber,SE_mutex);
unlock(carnumber,NE_mutex);
break;
case SOUTH:
lock(carnumber,NW_mutex);
lock(carnumber,SW_mutex);
printf("Car %d, Moving North-South\n\n", carnumber);
unlock(carnumber,SW_mutex);
unlock(carnumber,SW_mutex);
break;
case EAST:
lock(carnumber,SE_mutex);
lock(carnumber,SW_mutex);
printf("Car %d, Moving West-East\n\n", carnumber);
unlock(carnumber,SW_mutex);
unlock(carnumber,SE_mutex);
break;
case WEST:
lock(carnumber,NW_mutex);
lock(carnumber,NE_mutex);
printf("Car %d, Moving East-West\n\n", carnumber);
unlock(carnumber,NE_mutex);
unlock(carnumber,NW_mutex);
break;
}
}

static void turnleft(orientation_t cardirection, unsigned int carnumber)
{
switch(cardirection){
case NORTH:
lock(carnumber,NE_mutex);
lock(carnumber,SE_mutex);
lock(carnumber,SW_mutex);
printf("Car %d, Moving South-West\n\n", carnumber);
unlock(carnumber,SW_mutex);
unlock(carnumber,SE_mutex);
unlock(carnumber,NE_mutex);
break;
case SOUTH:
lock(carnumber,NW_mutex);
lock(carnumber,NE_mutex);
lock(carnumber,SW_mutex);
printf("Car %d, Moving North-East\n\n", carnumber);
unlock(carnumber,SW_mutex);
unlock(carnumber,NE_mutex);
unlock(carnumber,NW_mutex);
break;
case EAST:
lock(carnumber,NW_mutex);
lock(carnumber,SE_mutex);
lock(carnumber,SW_mutex);
printf("Car %d, Moving West-North\n\n", carnumber);
unlock(carnumber,SW_mutex);
unlock(carnumber,SE_mutex);
unlock(carnumber,NW_mutex);
break;
case WEST:
lock(carnumber,NW_mutex);
lock(carnumber,NE_mutex);
lock(carnumber,SE_mutex);
printf("Car %d, Moving East-South\n\n", carnumber);
unlock(carnumber,SE_mutex);
unlock(carnumber,NE_mutex);
unlock(carnumber,NW_mutex);
break;
}
}

static void turnright(orientation_t cardirection, unsigned int carnumber)
{
switch(cardirection){
case NORTH:
lock(carnumber,NE_mutex);
printf("Car %d, Moving South-East\n\n", carnumber);
unlock(carnumber,NE_mutex);
break;
case SOUTH:
lock(carnumber,SW_mutex);
printf("Car %d, Moving North-West\n\n", carnumber);
unlock(carnumber,SW_mutex);
break;
case EAST:
lock(carnumber,SE_mutex);
printf("Car %d, Moving West-South\n\n", carnumber);
unlock(carnumber,SE_mutex);
break;
case WEST:
lock(carnumber,NW_mutex);
printf("Car %d, Moving East-North\n\n", carnumber);
unlock(carnumber,NW_mutex);
break;
}
}

static void * approachintersection(void* arg){
unsigned int * carnumberptr;
unsigned int carnumber;
orientation_t cardir = (orientation_t)random()%4;
unsigned long turn = random()%3;

printf(" %d and %d \n",cardir,turn);

carnumberptr = (unsigned int*) arg;
carnumber = (unsigned int) *carnumberptr;

if(turn==LEFT){
turnleft(cardir, carnumber);
} else if(turn==RIGHT){
turnright(cardir, carnumber);
} else {
gostraight(cardir, carnumber);
}

return (void*)carnumberptr;
}

int main(int argc, char **argv){

int NUMCARS;
printf("enter NUMCARS\n");
scanf("%d",&NUMCARS);
if(NUMCARS<=10)
{
int index, tid;
unsigned int carids[NUMCARS];
pthread_t carthreads[NUMCARS];

for(index = 0; index <NUMCARS; index++)
{
carids[index] = index;
tid = pthread_create(&carthreads[index], NULL, approachintersection,(void*)&carids[index]);
printf("%d",tid);
}

for(index = 0; index <NUMCARS; index++){
pthread_join(carthreads[index], NULL);
}
}
else
{
printf("THE NUMBER OF NUMCARS ARE NOT VALID \n");
}
printf("Done\n");
return 1;
}
