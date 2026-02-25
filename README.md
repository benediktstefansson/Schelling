# The Schelling model of segregation in C

These pages describe and explain C programs which implement Thomas Schelling's model of segregation from Micromotives and Macrobehavior, also sometimes referred to as Schelling's tipping model. There are three versions of the C program, starting from a simple one and then adding some more features. 

I wrote this code over a quarter of a century ago, while at UCLA. The code follows closely the structure of a Pascal program of the same model by Robert Axelrod.

Assuming that you are compiling this on a Unix computer with the gcc compiler, issue the following statement:

gcc -g -lm -o program program.c

where program is either schelling,schelling2 or schelling3

## Annotated code for Schelling's segregation model in C: Version schelling.c

This is a web page with annotations for the code in the file shelling.c, which is a C program implementing Thomas Schelling's model of segregation from Micromotives and Macrobehavior, according to a Pascal version of that model written by Robert Axelrod.

### Global declarations

As with other C programs the file starts with #include statements, which instruct the compiler to import programming libraries that this program will need. What follows are definitions of constants, and declarations of global variables and function prototypes.

```C
#include <stdio.h>
#include <math.h>
#include <stdlib.h>

/* Schelling's model of segregation. C code by Benedikt Stefansson January 1998
   based on original Pascal code by Tom Belding and Robert Axelrod */

/* Define constants */
#define number_of_reports 8
#define events_per_report 100
#define num_white 25
#define num_agents 50

/* Declare arrays */
static int occupant[64];
static int color[num_agents];
static int location[num_agents];
static int neighbor_loc[64][8];


/* Declare subroutines */ 
int content(int i);
void random_move(int i);
int get_random_location();
void initial_output();
void initialize_agent_color();
void initialize_agent_location();
void initialize_neighbor_list();
void periodic_output(int event,int moves);
```

### The main function

The main function is the "main loop" of the program. It will set up the model according to the predefined parameters, and fill in the initial values for agents, such as color and location. Then it iterates through a two level loop, an outer loop which are the "time steps" and an inner loop which are the events within each time step. Each event consists of a function call to an agent, where it is determined whether the agent is satifsied or not, and if the agent is not happy in his neighborhood he finds a random empty location on the lattice and moves there.

```C
void main() {
  int i;
  int report;
  int events_in_report;
  int event=0;
  int moves_this_period=0;
  
  /* Initialize lattice and neighborhood list */
  initial_output();
  initialize_agent_color();
  initialize_agent_location();
  initialize_neighbor_list();
  periodic_output(0,0);

  /* Count timesteps or events */
  event = 0;
  i = 0;

  /* Outer event loop */
  for (report=0;report<number_of_reports;report++) {
    /* Count of actual moves */
    moves_this_period = 0;

    /* Inner event loop */
    for (events_in_report = 0;events_in_report < events_per_report;events_in_report++) {
      event++;
      /* Work our way through the agent list */
      if (i > num_agents)
	i = 0;
      if (!content(i)) {
	/* If agent is not content then move it*/
	moves_this_period++;
	random_move(i);
      }
      i++;
    }
    /* Print out report */
    periodic_output(event,moves_this_period);   
  } 

  exit(0);
}
```

### The content() function

The content function takes an integer argument and returns an integer value. If the return value is 0 the agent is unhappy and if it is 1 he is content. The integer argument refers to the ID number of the agent, this allows the function to look up his location and neighborhood by ID.

```C
int content(int i) {
  /* Calculate if agent i is content and
     return 1 if he is and 0 if not */
  int n;
  int result;
  int neighboor;
  int address;
  int neighborhood_cell;
  int same_color_count=0;
  int occupied_count=0;

  address=location[i];
 
  for(n=0;n<8;n++) {
    /* Check if neighborhood cell is on map and occupied */
    neighborhood_cell=neighbor_loc[address][n];
    if(neighborhood_cell!=-1 && occupant[neighborhood_cell]!=-1) {
      /* There is someone there */
      occupied_count++;
      /* But is he of the right color? */
      if(color[occupant[neighborhood_cell]]==color[i]) 
	same_color_count++;
    }
  }

  /* Content if more than 1/3 of neighbors are same color */
 if(same_color_count*3 > occupied_count)
   result=1;
 else
   result=0;

 return result;
}
```

### Functions to find random location

The random_move function takes an integer argument and returns void (nothing). The argument is the ID number of an agent. The function calls a random number generating function through the get_random_location() function, until it finds an empty cell. It then proceeds to put the agent into the empty sell, and updating the information in the system accordingly.

The get_random_location() function takes no argument and returns an integer between 0 and the lattice size (which is hardcoded as being 64, i.e. 8 by 8). It accomplishes this by calling the rand() function, which returns an integer between 0 and RAND_MAX. We divide by RAND_MAX and thus get a floating point number between 0 and 1. Multiply this number by the size of the lattice and you get a number between 0 and the lattice_size. Note the casting between variable types, first from int to float and then back to int.

```C
void random_move(int i) {    
      int trial_location;
      int look=0;

      /* Choose random locations until you
	 manage to find an empty one */
      do{
	trial_location=get_random_location();
      } while(occupant[trial_location]!=-1);

      /* OK - found empty location so move */
      /* First remove agent at old location */
      occupant[location[i]]=-1;
      /* Then add him at the new location */
      occupant[trial_location]=i;
      /* Finally update location info for agent */
      location[i]=trial_location;
     
}

int get_random_location() {
  float p;
  
  p=((float)rand()/(float)RAND_MAX);

  return (int)(p*64.0);
}
```

### Initialization routines

The functions initialize_agent_color() and initialize_agent_location() select the color and initial location of agents at runtime. The first num_white agents are made white (0) and the rest non white (1).

```C
void initialize_agent_color() {
  /* Give each agent a color 0 or 1 */
  int i;

  for(i=0;i<num_agents;i++) {
    if(i< num_white) 
      color[i]=0;
    else
      color[i]=1;
  }
}

void initialize_agent_location() {
  int i;
  int trial_location;

  for(i=0;i<64;i++) {
    occupant[i]=-1;
  }
  
  for(i=0;i<num_agents;i++) {
    do {
      trial_location=get_random_location();
    }while(occupant[trial_location]!=-1);

    occupant[trial_location]=i;
    location[i]=trial_location;
  }  

}
```

### Generation of neighborhoods

The function initialize_neighbor_list() fills in arrays with the location number of each of the 8 neighboring cells to each interior cell on the array - and the border cells which only have 5 neighbors. The calculation proceeds as follows: First calculate the default 8 cells for each of the 64 locations, then set the neighbors that are off the map to -1 which indicates that the location doesn't exist.

```C
void initialize_neighbor_list() {
  /* Precalculates Moore neighborhoods
     for each location on map */
  int l;

  for (l = 0; l < 64; l++) {
    /* First calculate general case */
    neighbor_loc[l][0] = l - 9; /* NW */
    neighbor_loc[l][1] = l - 8; /* N  */
    neighbor_loc[l][2] = l - 7; /* NE */
    neighbor_loc[l][3] = l - 1; /* W  */
    neighbor_loc[l][4] = l + 1; /* E  */
    neighbor_loc[l][5] = l + 7; /* SW */
    neighbor_loc[l][6] = l + 8; /* S  */
    neighbor_loc[l][7] = l + 9; /* SE */
    
    /* Correct for top row */
    if (l < 8) { 
      neighbor_loc[l][0] = 0;
      neighbor_loc[l][1] = 0;
      neighbor_loc[l][2] = 0;
    }
    /* Correct for bottom row */
    if (l > 55) { 
      neighbor_loc[l][5] = -1;
      neighbor_loc[l][6] = -1;
      neighbor_loc[l][7] = -1;
    }
    /* Correct for right side */
    if ((l % 7) == 0) {   
      neighbor_loc[l][2] = -1;
      neighbor_loc[l][4] = -1;
      neighbor_loc[l][7] = -1;
    }
    /* Correct for left side */
    if ((l % 8) == 0) { 
      neighbor_loc[l][0] = -1;
      neighbor_loc[l][3] = -1;
      neighbor_loc[l][5] = -1;
    }
  } 

}
```

### Output functions

Two functions to print a brief blurb at the beginning of a run, and then the periodic reports, which show the state of the lattice in terms of which agent occupies which cell and the color or each cell's occupant. The periodic_report() function is called after each events_per_period events.

```C
void initial_output() {
  /* Prints some information at beginning of run */

  printf("Schelling's segregation model, coded by Benedikt Stefansson\n
          based on Pascal program by by Robert Axelrod and Ted Belding\n");

  printf("Parameters:\n");
  printf("   Number of agents=%3d\n",num_agents);
  printf("   Number of agents with color 0=%3d\n",num_white);

}

void periodic_output(int event,int moves) {
  /* Prints out periodic state of map */
  int line;
  int address;
  int column;
    
  if(event==0) 
    printf("Initial state of map\n");
  else
    printf("Event %3d. Moves this period %3d\n",event,moves);  
  
  address=0;
  for(line=0;line<8;line++) {
    for(column=0;column<8;column++) {
      /* Output agent map */
      if(occupant[address]==-1) 
	printf(" . ");
      else 
	printf("%3d",occupant[address]);
      address++;
    }
    /* Rewind and put space between */
    address-=8;
    printf("     ");
    for(column=0;column<8;column++) {
      /* Output color map */
      if(occupant[address]==-1)
	printf(" . ");
      else {
	if(color[occupant[address]]==0)
	  printf(" @ ");
	if(color[occupant[address]]==1)
	  printf(" # ");
      }
      address++;
    }
    printf("\n");
  }
  printf("\n");

}
```

## Annotated code for schelling2.c

Annotated code for Schelling's segregation model in C: Version schelling2.c

This annotation only mentions the differences between schelling.c and schelling2.c.

### Global declarations

The global declarations for this program are different from <a
href="schelling_annotated.html">schelling.c</a>, since we have made some
constants into parameters which can be determined at runtime and added
new functions to deal with that.

Note in particular the statements regarding global variables, the
arrays and then the new functions get_parameters() and
initialize_arrays()

```C
#include <stdio.h>
#include <math.h>
#include <stdlib.h>

/* Schelling's model of segregation. C code by Benedikt Stefansson January 1998
   based on original Pascal code by Tom Belding and Robert Axelrod */

/* Global vars */
int number_of_reports=8;
int events_per_report=100;
int num_white=25;
int num_agents=50;

/* Declare arrays */
static int occupant[64];
static int neighbor_loc[64][8];
static int * color;
static int * location;


/* Declare subroutines */ 
int content(int i);
void random_move(int i);
int get_random_location();
void initial_output();
void initialize_agent_color();
void initialize_agent_location();
void initialize_neighbor_list();
void periodic_output(int event,int moves);
void get_parameters();
void initialize_arrays();
```


### The main function

The main difference between the two main functions in this program
and in  <a href="schelling_annotated.html">schelling.c</a> is that here
we call get_parameters() and initialize_variables() before starting
the simulation.

```C
void main() {
  int i;
  int report;
  int events_in_report;
  int event=0;
  int moves_this_period=0;
  
  /* Get parameter settings */
  get_parameters();

  /* Allocate memory for agent lists */
  initialize_arrays();
  
  /* Initialize lattice and neighborhood list */
  initial_output();
  initialize_agent_color();
  initialize_agent_location();
  initialize_neighbor_list();
  periodic_output(0,0);

  /* Count timesteps or events */
  event = 0;
  i = 0;

  /* Outer event loop */
  for (report=0;report<number_of_reports;report++) {
    /* Count of actual moves */
    moves_this_period = 0;

    /* Inner event loop */
    for (events_in_report = 0;events_in_report < events_per_report;events_in_report++) {
      event++;
      /* Work our way through the agent list */
      if (i >= num_agents)
	i = 0;
      if (!content(i)) {
	/* If agent is not content then move it*/
	moves_this_period++;
	random_move(i);
      }
      i++;
    }
    /* Print out report */
    periodic_output(event,moves_this_period);   
  } 

  exit(0);
}
```

### Changes to function periodic_output()

We have made slight changes to the function periodic_output(), in
that the strings used to symbolize the different colors on the board,
and an emtpy cell, are now stored in a char array. The changes are to
be found in the initialization of the local variables and in the print
loop. Compare this to the previous version.

```C
void periodic_output(int event,int moves) {
  /* Prints out periodic state of map */
  int line;
  int address;
  int column;
  char str[3]={'.','@','#'};
    
  if(event==0) 
    printf("Initial state of map\n");
  else
    printf("Event %3d. Moves this period %3d\n",event,moves);  
  
  address=0;
  for(line=0;line<8;line++) {
    for(column=0;column<8;column++) {
      /* Output agent map */
      if(occupant[address]==-1) 
	printf(" . ");
      else 
	printf("%3d",occupant[address]);
      address++;
    }
    /* Rewind and put space between */
    address-=8;
    printf("     ");
    for(column=0;column<8;column++) {
      /* Output color map */
      if(occupant[address]==-1)
	printf(" %c ",str[0]);
      else 
	printf(" %c ",str[color[occupant[address]]+1]);
      address++;
    }
    printf("\n");
  }
  printf("\n");

}
```

### Input of parameters

The program now prints a blurb and then queries the user for
several parameters, number of agents, number of white type and the
number of reports and events per report.

The input of parameters follows a simple format. We use the
variable temp to hold the input, and if temp is not in the
proper interval we try again. The variable is thus set to 0 (FALSE)
before each query, and the while loop will continue to be executed
until a correct value is input. Then the temp value is passed to the
corresponding parameter.

```C
void get_parameters() {
  /* Read in several parameters before running sim */
  int temp=0;

  printf("Schelling's segregation model\n");
  printf("code by Benedikt Stefansson\n");
  printf("based on Pascal program by\n");
  printf("Robert Axelrod and Ted Belding\n");

  while(!temp) {
    printf("Input number of agents (0-64): ");
    scanf("%d",&temp);
    if(temp<=0 || temp>=64) {
      printf("Number of agents must be between 0-64\n");
      temp=0;
    } else {
      num_agents=temp;
    }
  }
  temp=0;
  
  while(!temp) {
    printf("Number of white agents (1-%d): ",num_agents-1);
    scanf("%d",&temp);
    if(temp<=0 || temp>=num_agents) {
      printf("Number of white agents must be between 1-%d\n",num_agents);
      temp=0;
    } else {
      num_white=temp;
    }
  }
  temp=0;
  
  while(!temp) {
    printf("Number of reports to print: ");
    scanf("%d",&temp);
    if(temp<=0) {
      printf("Number of reports must be positive\n");
      temp=0;
    } else {
      number_of_reports=temp;
    }
  }
  temp=0;
  
  while(!temp) {
    printf("Number of events per report: ");
    scanf("%d",&temp);
    if(temp<=0) {
      printf("Number of events must be positive\n");
      temp=0;
    } else {
      events_per_report=temp;
    }
  }

}
```

### Dynamic allocation of arrays

Since the user is free to choose the number of agents at runtime,
we now dynamically allocate the color and location arrays. The library
calloc() is used and the address of the first element of the resulting
arrays is stored in the pointers color and location respectively.

```C
void initialize_arrays() {
  /* Allocate memory for the color and location arrays */
  color=calloc(num_agents,sizeof(int));
  location=calloc(num_agents,sizeof(int));

}
```
