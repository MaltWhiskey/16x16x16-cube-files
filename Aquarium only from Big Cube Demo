#include <math.h>
#include <exception>
#include "application.h"

#define PIXEL_PIN D0
#define totalPIXEL 4096
#define stripPIXEL totalPIXEL/8    //THERE IS FOUR PINS TO DRIVE EACH STRIPS
#define PIXEL_TYPE WS2812B

#define SIZE 16
#define PI 3.14159
#define NUM_BUB 5


/**********************************
 * flip variables *
 * ********************************/
 //accelerometer pinout
#define X A0
#define Y A1
#define Z A6
#define NUM_SAMPLES 100
#define MICROPHONE A7
#define GAIN_CONTROL D5
int accelerometer[3];
unsigned long totals[3];
int runningAverage[3];
int maxBrightness=50;
boolean whack[3];
boolean whacked=false;
uint8_t PIXEL_RGB[totalPIXEL*3];  //EACH PIXELS HAS 3 BYTES DATA

#define WHACK_X 15
#define WHACK_Y 15
#define WHACK_Z 15

/*  datatype definitions
*/

/**   An RGB color. */
struct Color
{
  unsigned char red, green, blue;

  Color(int r, int g, int b) : red(r), green(g), blue(b) {}
  Color() : red(0), green(0), blue(0) {}
};

Color bodyColor(207, 121 ,8);
Color tailColor(207, 60, 8);
Color bubbleColor(107, 102, 226);
/**   A point in 3D space.  */
struct Point
{
  float x;
  float y;
  float z;
  Point() : x(0), y(0), z(0) {}
  Point(float _x, float _y, float _z) : x(_x), y(_y), z(_z) {}

  Point& operator=(const Point &a)
  {
	//Defines the = operator to be used between two different structures.
	x=a.x;
	y=a.y;
	z=a.z;
	return *this;
  }

  void normalize()
  {
	//Normalize function, this decreases an axis allowing it to be manipulated on a smaller scale.
	float v;
	v=sqrt(pow(x, 2)+pow(y, 2)+pow(z, 2));
	x=x/v;
	y=y/v;
	z=z/v;
  }

};
int timeout=0;
Point foodPos;
boolean foodPresent=false;
void setVoxel(int x, int y, int z, Color col);
void setVoxel(Point p, Color col);
Color getVoxel(int x, int y, int z);
Color getVoxel(Point p);
void line(Point p1, Point p2, Color col);
void line(int x1, int y1, int z1, int x2, int y2, int z2, Color col);
void sphere(Point center, float radius, Color col);
void sphere(Point center, float radius, Color col, int res);
void sphere(float x, float y, float z, float radius, Color col);
void sphere(float x, float y, float z, float radius, Color col, int res);
void background(Color col);
Color colorMap(float val, float minVal, float maxVal);
Color lerpColor(Color a, Color b, int val, int minVal, int maxVal);
Point add(Point a, Point b);
Point crossMultiply(Point o, Point p);
void updateAccelerometer();
void initAccelerometer();
float distance(Point a, Point b);
void newBubble();
void bubbleFloat();
void printPoint(Point p);
struct Fish
{
	/*
		Here is the fish structure, where the fish is created.
	*/
	Point center; //The various vertices required to make the fish. 
	Point Hinge;
	Point TopTip;
	Point BottomTip;
	Point direction;
	Point Up;
	Point side;
	Point turnRadius;

	float Tip;
	float height=3;
	float width=3;
	float radius=2;
	float theta=0;
	float rad[2];
	int count;
	float angle;
	float speed=.1;
	float enthusiasm=8;
	int dinnerTime=10;
	int eatingTime=0;
	float sweep=.4;
	bool eating=false;
	
	Fish(Point c, Point d) 
	{					   
		//Fish Constructor, defining it for two Points.	
	
		center=c;		
		direction=d;
		hitDetection();
		setDirection();
	}
	Fish(float a, float b, float c, Point d)
	{
		//Fish constructor defining for the x, y, z starting position and the Point Direction
		direction=d;
		center={a, b, c};
		hitDetection();
		setDirection();
		
	}
	Fish( float a, float b, float c, float d, float e, float f)
	{
		//Fish constructor defining for the x, y, z starting position and the x, y, z of the direction it is facing.
		center={a, b, c};
		direction={d, e, f};
		hitDetection();
		setDirection();
	}
	Fish(Point a, float b, float c, float d)
	{
		//Fish constructor defining for the Point starting position and the x, y z of the direction.
		center=a;
		direction={b, c, d};
		hitDetection();
		setDirection();
	}
	Fish()
	{
		//Default constructor, presets the center position and the direction it is facing
		center={7,7,7};
		direction={1,0,0};
		hitDetection();
		setDirection();
	}
	void display()	//Display draws the fish, and it also calls hitDetect and setDirection functions
	{
		hitDetection();
		sphere(center, radius, bodyColor);
		line(Hinge, TopTip, tailColor);
		line(TopTip, BottomTip, tailColor);
		line(Hinge, BottomTip, tailColor);	
		//setVoxel(Hinge, Color(0, 0, 255));
		//This is what causes the fish to move. The current direction plus it's center position. 
		center={ center.x + direction.x*speed,center.y + direction.y*speed,center.z + direction.z*speed};
		setDirection();
	}

	void hitDetection()
	{
		//This function detects the boundaries on the walls, and it also defines what the fish does when
		//there is fish food.
		if (foodPresent)
		{
			/*
				When food is present, change the direction of the fish towards the food, and
				the fish starts moving into that directions.
			*/
			direction=foodPos;	
			direction={direction.x-center.x, direction.y-center.y, direction.z-center.z};  //point the fish to the food   
			direction.normalize();
			enthusiasm=16;
			speed=.4;
			if (distance(center,foodPos)<=radius+2)
			{	
				//Once the fish's encompasses the food it will then wait until eating time is
				//greater than dinner time which it will then set everything back to normal.
				//setDirection();
				eatingTime++;
				if (eatingTime>dinnerTime)
				{
					
				
					eating=false;
					foodPresent=false;	
					enthusiasm=8;
					speed=.1;
					hitDetection();
					eatingTime=0;
					}
				}
		}
		if (center.x>=15 ||center.x<=0)
		{
			//These are the boundaries of the cube
			if (angle<20) //The angle the fish well move in.
			{  
				rotate3D('x', 20);//Calls the Rotate function
			} 
			else
			{
				//The count is to make sure that the fish rotates
				//back into the cube, and also having direction.x increment
				//to prevent it from getting stuck on a border.
				count++;
				if (center.x>=15)
					direction.x-=1;
				else if (center.x<=0)
					direction.x+=1;
				if (count==2)
				{
					//Once count equals 2 it will stop the rotation.
					angle=0;
					count=0;
				}
			}
		} 
		else if (center.y>=15 ||center.y<=0)
		{
			
			if (angle<20)
			{  
				rotate3D('y', 20);
			} 
			else 
			{
				count++;
				if (center.y>=15)
					direction.y-=1;
				else if (center.y<=0)
					direction.y+=1;
				if (count==2)
				{
					angle=0;
					count=0;
				}
			}	
		} 
	else if (center.z>=15||center.z<=0)
	{
	
		if (angle<20)
		{  
			rotate3D('z', 20);
		} 
		else
		{
			count++;

			if (center.z>=15)
				direction.z-=1;
			else if (center.z<=0)
				direction.z+=1;
		}
		if (count==2)
		{
			angle=0;
			count=0;
		}
		
	}
	
	/*
		These if statements are to help keep the fish inside the cube.
		There are also similar if statement inside the earlier if statements, but those
		do not help much when the fish goes into a corner between two of the axes.
	*/
	if (center.x>15)
		direction.x-=1;
	else if (center.x<=2)
		direction.x+=1;
	if (center.y >=15)
		direction.y-=1;
	else if (center.y<=0)
		direction.y +=1;
	if (center.z>=15)
		direction.z -=1;
	else if (center.z<=0)
		direction.z+=1; 		
	}
	void setDirection()
	{
		/*Set directions is what calculates the fishes movements, and the
		movements of the tail. 
		*/
		theta+=.1 *enthusiasm;//Enthusiasm allows us to manipulate how fast the fish's tail moves
		//Calculating the different axes based off of the center and the x-axis.
		//Up is the y axis, and side is the z-axis. 
		Up=crossMultiply(direction, center);
		Up.normalize();
		side=crossMultiply(direction, Up);
		direction.normalize();
		//Sweep is the sweep of the tail, moves. 
		sweep=sweep*(float)sin(theta);
			
		side={side.x *sweep, side.y*sweep, side.z*sweep};
		//Up is being multiplied by the height of the tail to help with calculating the tips.
		Up={Up.x*height, Up.y*height, Up.z*height};
		
		//Hinge is where the tail connects to the body.
		Hinge={center.x -(direction.x *radius),center.y -(direction.y *radius), center.z -(direction.z *radius)};
		//Direction is being multiplied by the width of the tail to help with calculating the tips.
		direction={direction.x*width, direction.y*width, direction.z*width};
		
		
		TopTip={Hinge.x + Up.x-direction.x+side.x, Hinge.y+Up.y-direction.y+side.y, Hinge.z+Up.z-direction.z+side.z};
		BottomTip={Hinge.x-Up.x-direction.x+side.x, Hinge.y-Up.y-direction.y+side.y, Hinge.z-Up.z-direction.z+side.z};
		
	}
	
	void rotate3D(char a, float b)
	{
		/*
			This function takes in an axis and an angle the fish will rotate.
			Each of the if statements is the math formula to rotate along the specific axis they distinguish.
			It is then added directly into the direction vortex. 
		*/
		float temp[2];
		if (a=='x')
		{
			//Rotating along the X axis formula:
			temp[0]= (float)cos(b)-(float)sin(b);
			temp[1]= (float) sin(b)+(float)cos(b);
			
			
			direction.y+=temp[0];
			direction.z+=temp[1];
			
		}
		if (a=='y')
		{
			temp[0]= (float) cos(b)+(float)sin(b);
			temp[1]= (float)sin(b)+(float)cos(b);
			direction.x+=temp[0];
			direction.z+=temp[1];
		}
		if (a=='z')
		{
			temp[0]= (float)cos(b)-(float)sin(b);
			temp[1]= (float)sin(b)+ (float)cos(b);
			direction.x+=temp[0];
			direction.y+=temp[1];
		} 
		//This variable called angle is used to help exit a side, basically to help stop the fish 
		//from being unable to move along.
		angle+=abs(temp[0])+abs(temp[1]);
	}
	void updateDirection(float a, float b, float c)
	{
		//A function to update direction outside of the fish structure.
		direction={a, b, c};
	
	}

};


Fish fish;
Point bubble[NUM_BUB];
float speed[NUM_BUB];
int ending[NUM_BUB];
int incomingByte;
void setup() 
{
	for(int i=0;i<NUM_BUB;i++)
		newBubble(i);
	incomingByte=0;
    randomSeed(analogRead(A0));
    Serial.begin(115200);
	pinMode(GAIN_CONTROL, OUTPUT);
  	digitalWrite(GAIN_CONTROL, LOW);
    pinMode(D0,OUTPUT);  //PB7
    pinMode(D1,OUTPUT); //PB6
    pinMode(D2,OUTPUT);  //BP5
    pinMode(D3,OUTPUT);  //PB4
    pinMode(A2,OUTPUT);  //PA4
    pinMode(A3,OUTPUT); //PA5
    pinMode(A4,OUTPUT);  //BA6
    pinMode(A5,OUTPUT);  //PA7
	initAccelerometer();
    pinMode(D7, OUTPUT);
    digitalWrite(D7, HIGH);
	//Defines a fish object
	fish= Fish(8, 8, 8,random(-1,1),1,random(-1,1));
	
}
void loop()
{
	
	background(Color(0,0,0));

	if((whacked)&&((millis()-timeout)>250))
	{
		//When the cube motion sensor senses something
		//food is true.
		foodPresent=true;
		
		//setVoxel(7,15,7, Color(255,100,0));
		feed();
		timeout=millis();
	}
	if(foodPresent)
	{
		setVoxel(foodPos, Color(255,255,0));
	}
	updateAccelerometer();
	fish.display();
	bubbleFloat();
	show();
}
  Point crossMultiply(Point o, Point p)
  {
	Point q;
	float a= o.y*p.z - o.z*p.y;
	float b=o.z*p.x-o.x*p.z;
	float c=o.x*p.y-o.y*p.x;
	q={a,b,c};
	
	return q;
  }

void background(Color col)
{
    for(int x=0;x<SIZE;x++)
        for(int y=0;y<SIZE;y++)
            for(int z=0;z<SIZE;z++)
                setVoxel(x,y,z,col);
}

void setVoxel(int x, int y, int z, Color col)
{
    if((x>=0)&&(x<SIZE))
        if((y>=0)&&(y<SIZE))
            if((z>=0)&&(z<SIZE))
            {
                int index=z*256+x*16+y;
                PIXEL_RGB[index*3]=col.green;
                PIXEL_RGB[index*3+1]=col.red;
                PIXEL_RGB[index*3+2]=col.blue;
            }
}

void setVoxel(Point p, Color col)
{
    setVoxel(p.x, p.y, p.z, col);
}

void line(Point p1, Point p2, Color col)
{
    line(p1.x, p1.y, p1.z, p2.x, p2.y, p2.z, col);
}

void line(int x1, int y1, int z1, int x2, int y2, int z2, Color col)
{
  Point currentPoint;
  currentPoint.x=x1;
  currentPoint.y=y1;
  currentPoint.z=z1;
  
  int dx = x2 - x1;
  int dy = y2 - y1;
  int dz = z2 - z1;
  int x_inc = (dx < 0) ? -1 : 1;
  int l = abs(dx);
  int y_inc = (dy < 0) ? -1 : 1;
  int m = abs(dy);
  int z_inc = (dz < 0) ? -1 : 1;
  int n = abs(dz);
  int dx2 = l << 1;
  int dy2 = m << 1;
  int dz2 = n << 1;

  if((l >= m) && (l >= n)) {
    int err_1 = dy2 - l;
    int err_2 = dz2 - l;

    for(int i = 0; i < l; i++) {
      setVoxel(currentPoint, col);

      if(err_1 > 0) {
        currentPoint.y += y_inc;
        err_1 -= dx2;
      }

      if(err_2 > 0) {
        currentPoint.z += z_inc;
        err_2 -= dx2;
      }

      err_1 += dy2;
      err_2 += dz2;
      currentPoint.x += x_inc;
    }
  } else if((m >= l) && (m >= n)) {
    int err_1 = dx2 - m;
    int err_2 = dz2 - m;

    for(int i = 0; i < m; i++) {
      setVoxel(currentPoint, col);

      if(err_1 > 0) {
        currentPoint.x += x_inc;
        err_1 -= dy2;
      }

      if(err_2 > 0) {
        currentPoint.z += z_inc;
        err_2 -= dy2;
      }

      err_1 += dx2;
      err_2 += dz2;
      currentPoint.y += y_inc;
    }
  } else {
    int err_1 = dy2 - n;
    int err_2 = dx2 - n;

    for(int i = 0; i < n; i++) {
      setVoxel(currentPoint, col);

      if(err_1 > 0) {
        currentPoint.y += y_inc;
        err_1 -= dz2;
      }

      if(err_2 > 0) {
        currentPoint.x += x_inc;
        err_2 -= dz2;
      }

      err_1 += dy2;
      err_2 += dx2;
      currentPoint.z += z_inc;
    }
  }

  setVoxel(currentPoint, col);
}


Point add(Point a, Point b)
{
	return Point(a.x+b.x, a.y+b.y, a.z+b.z);
}

// draws a hollow  centered around the 'center' Point, with radius
// radius and color col
void sphere(Point center, float radius, Color col) {
     float res = 30;
     for (float m = 0; m < res; m++)
     	 for (float n = 0; n < res; n++)
	     setVoxel(center.x + radius * sin((float) PI * m / res) * cos((float) 2 * PI * n / res),
		      center.y + radius * sin((float) PI * m / res) * sin((float) 2 * PI * n / res),
		      center.z + radius * cos((float) PI * m / res),
		      col);
}

// draws a hollow  centered around the 'center' Point, with radius
// radius and color col
void sphere(Point center, float radius, Color col, int res) {
     for (float m = 0; m < res; m++)
     	 for (float n = 0; n < res; n++)
	     setVoxel(center.x + radius * sin((float) PI * m / res) * cos((float) 2 * PI * n / res),
		      center.y + radius * sin((float) PI * m / res) * sin((float) 2 * PI * n / res),
		      center.z + radius * cos((float) PI * m / res),
		      col);
}

void sphere(float x, float y, float z, float radius, Color col) {
     sphere(Point(x,y,z),radius, col);
}

void sphere(float x, float y, float z, float radius, Color col, int res) {
     sphere(Point(x,y,z),radius, col, res);
}

void show(){
    uint8_t *ptrA,*ptrB,*ptrC,*ptrD,*ptrE,*ptrF,*ptrG,*ptrH;
    uint8_t mask;
    uint8_t c=0,a=0,b=0,j=0;
    
    GPIOA->BSRRH=0xE0;    //set A3~A5 to low
    GPIOB->BSRRH=0xF0;    //set D0~D4 to low
    GPIOC->BSRRH=0x04;     //set A2 to low
    
    ptrA=&PIXEL_RGB[0];
    ptrB=ptrA+stripPIXEL*3;
    ptrC=ptrB+stripPIXEL*3;
    ptrD=ptrC+stripPIXEL*3;
    ptrE=ptrD+stripPIXEL*3;
    ptrF=ptrE+stripPIXEL*3;
    ptrG=ptrF+stripPIXEL*3;
    ptrH=ptrG+stripPIXEL*3;
    
    delay(1);
    __disable_irq();

    uint16_t i=stripPIXEL*3;   //3 BYTES = 1 PIXEL
    
    while(i) { // While bytes left... (3 bytes = 1 pixel)
      i--;
      mask = 0x80; // reset the mask
      j=0;
        // reset the 8-bit counter
      do {
        a=0;
        b=0;
        c=0;

//========Set D0~D4, i.e. B7~B4=======
        if ((*ptrA)&mask) b|=0x10;// if masked bit is high
    //    else "nop";
        b<<=1;
        if ((*ptrB)&mask) b|=0x10;// if masked bit is high
    //    else "nop";
        b<<=1;
        if ((*ptrC)&mask) b|=0x10;// if masked bit is high
    //    else "nop";
        b<<=1;
        if ((*ptrD)&mask) b|=0x10;// if masked bit is high
   //     else "nop";

//=========Set A2, i.g. C2==========
        if ((*ptrE)&mask) c|=0x04;// if masked bit is high
   //     else "nop";
   
        GPIOA->BSRRL=0xE0;    //set A3~A5 to high
        GPIOB->BSRRL=0xF0;    //set D0~D4 to high
        GPIOC->BSRRL=0x04;    //set A2 to high
        

//=========Set A3~A5, i.e. A5~A7========   
        if ((*ptrF)&mask) a|=0x80;// if masked bit is high
        // else "nop";
        a>>=1;
        if ((*ptrG)&mask) a|=0x80;// if masked bit is high
   //     else "nop";
        a>>=1;
        if ((*ptrH)&mask) a|=0x80;// if masked bit is high

        a=(~a)&0xE0;
        b=(~b)&0xF0;
        c=(~c)&0x04;
        GPIOA->BSRRH=a;
        GPIOB->BSRRH=b;
        GPIOC->BSRRH=c;
        mask>>=1;
         asm volatile(
            "mov r0, r0" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t" "nop" "\n\t"
            ::: "r0", "cc", "memory");
        GPIOA->BSRRH=0xE0;    //set all to low
        GPIOB->BSRRH=0xF0;    //set all to low
        GPIOC->BSRRH=0x04;    //set all to low        
          // WS2812 spec             700ns HIGH
          // Adafruit on Arduino    (meas. 812ns)
          // This lib on Spark Core (meas. 792ns)
          /*
        if(j<7) { 
          asm volatile(
            "mov r0, r0" "\n\t" 
            ::: "r0", "cc", "memory");
        }
        */
        
      } while ( ++j < 8 ); // ...one color on a pixel done
      ptrA++;
      ptrB++;
      ptrC++;
      ptrD++;
      ptrE++;
      ptrF++;
      ptrG++;
      ptrH++;
    } // end while(i) ... no more pixels
    __enable_irq();    
}



/** Map a value into a color.
  The set of colors fades from blue to green to red and back again.

  @param val Value to map into a color.
  @param minVal Minimum value that val will take.
  @param maxVal Maximum value that val will take.

  @return Color from value.
*/
Color colorMap(float val, float minVal, float maxVal)
{
  const float range = 1024;
  val = range * (val-minVal) / (maxVal-minVal);

  Color colors[6];

  colors[0].red = 0;
  colors[0].green = 0;
  colors[0].blue = maxBrightness;

  colors[1].red = 0;
  colors[1].green = maxBrightness;
  colors[1].blue = maxBrightness;

  colors[2].red = 0;
  colors[2].green = maxBrightness;
  colors[2].blue = 0;

  colors[3].red = maxBrightness;
  colors[3].green = maxBrightness;
  colors[3].blue = 0;

  colors[4].red = maxBrightness;
  colors[4].green = 0;
  colors[4].blue = 0;

  colors[5].red = maxBrightness;
  colors[5].green = 0;
  colors[5].blue = maxBrightness;

  if(val <= range/6)
    return lerpColor(colors[0], colors[1], val, 0, range/6);
  else if(val <= 2 * range / 6)
    return(lerpColor(colors[1], colors[2], val, range / 6, 2 * range / 6));
  else if(val <= 3 * range / 6)
    return(lerpColor(colors[2], colors[3], val, 2 * range / 6, 3*range / 6));
  else if(val <= 4 * range / 6)
    return(lerpColor(colors[3], colors[4], val, 3 * range / 6, 4*range / 6));
  else if(val <= 5 * range / 6)
    return(lerpColor(colors[4], colors[5], val, 4 * range / 6, 5*range / 6));
  else
    return(lerpColor(colors[5], colors[0], val, 5 * range / 6, range));
}

/** Linear interpolation between colors.

  @param a, b The colors to interpolate between.
  @param val Position on the line between color a and color b.
  When equal to min the output is color a, and when equal to max the output is color b.
  @param minVal Minimum value that val will take.
  @param maxVal Maximum value that val will take.

  @return Color between colors a and b.
*/
Color lerpColor(Color a, Color b, int val, int minVal, int maxVal)
{
  int red = a.red + (b.red-a.red) * (val-minVal) / (maxVal-minVal);
  int green = a.green + (b.green-a.green) * (val-minVal) / (maxVal-minVal);
  int blue = a.blue + (b.blue-a.blue) * (val-minVal) / (maxVal-minVal);

  return Color(red, green, blue);
}
void initAccelerometer()
{
	runningAverage[0]=analogRead(X);
	runningAverage[1]=analogRead(Y);
	runningAverage[2]=analogRead(Z);
}
void feed()
{
		foodPos={random(3, 11), 15, random(3, 11)};
		setVoxel(foodPos, Color(255,255,0));
}
void updateAccelerometer()
{
	accelerometer[0]=analogRead(X);
	accelerometer[1]=analogRead(Y);
	accelerometer[2]=analogRead(Z);
	for(int i=0;i<3;i++)
	{
		totals[i]+=accelerometer[i];	
		//sweet running average algorithm:  average[i+1]=average[i]+(sample[i]-average[i])/NUM_SAMPLES
		//I average over 100 samples, or ~2.5 seconds
		runningAverage[i]=runningAverage[i]+((accelerometer[i]-runningAverage[i])/NUM_SAMPLES);
		whack[i]=false;
	}	
	if(abs(accelerometer[0]-runningAverage[0])>WHACK_X)
		whack[0]=true;
	if(abs(accelerometer[1]-runningAverage[1])>WHACK_Y)
		whack[1]=true;
	if(abs(accelerometer[2]-runningAverage[2])>WHACK_Z)
		whack[2]=true;
	whacked=whack[0] | whack[1] | whack[2];	
}
float distance(Point a, Point b)
{
	//A function that calculates the distance between two points.
	return(float)sqrt(pow(a.x-b.x, 2) + pow(a.y-b.y, 2) +pow(a.z-b.z, 2));
  
}
void newBubble(int i)
{
		bubble[i]={random(SIZE), random(-10, 0), random(SIZE)};
		speed[i]=(float) random(100, 300)/1000;
		ending[i]=random(19, 24);
}
void bubbleFloat()
{
	for(int i=0; i<NUM_BUB;i++)
	{
			setVoxel(bubble[i], bubbleColor);
			bubble[i].y+=speed[i];
			if(bubble[i].y>ending[i])
				newBubble(i);
	}
	Serial.println();
}

void printPoint(Point p)
{
	Serial.print(p.x);
	Serial.print(", ");
	Serial.print(p.y);
	Serial.print(", ");
	Serial.println(p.z);
}
