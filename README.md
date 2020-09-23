<div align="center">

## Gravity Works\!


</div>

### Description

Gravity Simulator that uses the OpenGL Libraries. I used the GLUT event handler interface to control Mouse functions. Found at: http://modzer0.cs.uaf.edu/~hartsock/C_Cpp/OpenGL/Gravity.html
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Found on the World Wide Web](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/found-on-the-world-wide-web.md)
**Level**          |Beginner
**User Rating**    |5.0 (20 globes from 4 users)
**Compatibility**  |C, C\+\+ \(general\)
**Category**       |[Graphics/ Sound](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/graphics-sound__3-15.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/found-on-the-world-wide-web-gravity-works__3-32/archive/master.zip)





### Source Code

```
/************************************************************************\
|
|	Shawn R. Hartsock        Gravity.cpp
|	CS 381              Uses STL
|	Prof: Hartman
|	due: 12/16/1998
|
		Simulates Gravity. See PreDefined Menu... My favorite is to set
		collisions to absorbative and choose explode from the predef. menu
|		Keys : z,x,i,j,k,l,
|  Point: Drag to move particle points
|
\************************************************************************/
# include <vector>
# include <iterator>
# include <algorithm>
# include <GL/glut.h>
# include <math.h>
# include <stdlib.h>
using namespace std;
double M_PI = 3.14159265359; /*for Win32*/
/****************************************************************\
						Platform Independant Code starts Here:
\****************************************************************/
/****************************************************\
	Custom classes:
\********************/
struct particle{
	double x,y, mass, ax,ay, radius;
	int color;
	bool operator==(const particle &RHS){		return(this->mass == RHS.mass);	}
	bool operator!=(const struct particle RHS){	return(this->mass != RHS.mass);	}
	bool operator<(const particle &RHS){		return(this->mass < RHS.mass);	}
	bool operator<=(const particle &RHS){		return(this->mass <= RHS.mass);	}
	bool operator>(const particle &RHS){		return(this->mass > RHS.mass);	}
	bool operator>=(const particle &RHS){		return(this->mass >= RHS.mass);	}
	void operator++(){ this->mass += 1.0; }
}; // particle is just a data container
bool operator==(const particle &LHS, const particle &RHS){
	return(LHS.mass == RHS.mass);
}
bool operator!=(const particle &LHS, const particle &RHS){
	return(LHS.mass != RHS.mass);
}
bool operator<(const particle &LHS, const particle &RHS){
	return(LHS.mass < RHS.mass);
}
bool operator<=(const particle &LHS, const particle &RHS){
	return(LHS.mass <= RHS.mass);
}
bool operator>(const particle &LHS, const particle &RHS){
	return(LHS.mass > RHS.mass);
}
bool operator>=(const particle &LHS, const particle &RHS){
	return(LHS.mass >= RHS.mass);
}
/****************************************************\
	Global Variables...
\********************/
double	G = -9.7, ax,ay,nx,ny; /* 10,000 km^3 / 100 metric-tonne * sec^2 */
/** Universal Gravitational Constant 6.67E-8 cm**3/(gm*sec**2) **/
vector<particle> planets;
int current = 0;
int next  = 0;
int width = 200, height = 200;
particle temp_planet;
int colision = 1, sticky = 0, absorb = 0, refresh = 0, pause = 1,
newplanet = 0, set = 0, explode = 0, fountain = 0, stream = 0,
line = 0, space = 0, showplanets = 1;
double scale = 1.00, scw = 1.00, sch = 1.00;
double T_Y = 0, T_X = 0;
enum menu_options {
	CNONE, ELASTIC, ABSORB, FNONE, LINES, SPACE, CLEAR,
	PAUSE, ORBIT, RANDOM, EXPLODE, FOUNT,
	ZOOM, ZOOMUP, ZOOMDOWN, CENTER, MASSI, MASSD,
	NADA, STICKY, STREAM,
	QUIT
};
void circle(double x, double y, double radius, int color) {
	glPushMatrix();
	glTranslated(x+0.5,y+0.5,0);
	if(radius < 1.0){ radius = 1.0;     }
	glScalef(radius,radius,1.0);
	glCallList(color);
	glPopMatrix();
}
void draw_circle(){
	glNewList(1,GL_COMPILE);
	glTranslated(-0.5,-0.5,0);
	glBegin(GL_TRIANGLE_FAN);
	glColor3d(1.0, 1.0, 0.8);
	glVertex2d(0,0);
	glColor3d(0.5, 0.3, 0.0);
		for(double ii=0; ii <7; ii+=0.1){
			glVertex2d(cos(ii)+0.5,sin(ii)+0.5);
		}
	glColor3d(1.0, 1.0, 0.8);
	glVertex2d(0,0);
	glEnd();
	glEndList();
	glNewList(2,GL_COMPILE);
	glTranslated(-0.5,-0.5,0);
	glBegin(GL_TRIANGLE_FAN);
	glColor3d(0.9, 1.0, 1.0);
	glVertex2d(0,0);
	glColor3d(0.0, 0.1, 0.5);
		for(ii=0; ii <7; ii+=0.1){
			glVertex2d(cos(ii)+0.5,sin(ii)+0.5);
		}
	glColor3d(0.9, 1.0, 1.0);
	glVertex2d(0,0);
	glEnd();
	glEndList();
	glNewList(3,GL_COMPILE);
	glTranslated(-0.5,-0.5,0);
	glBegin(GL_TRIANGLE_FAN);
	glColor3d(1.0, 0.7, 1.0);
	glVertex2d(0,0);
	glColor3d(0.5, 0.0, 0.2);
		for(ii=0; ii <7; ii+=0.1){
			glVertex2d(cos(ii)+0.5,sin(ii)+0.5);
		}
	glColor3d(1.0, 0.7, 1.0);
	glVertex2d(0,0);
	glEnd();
	glEndList();
}
void draw_planets(){
	int rad = 3, size = planets.size();
	for(int ii=0; ii<size; ii++){
		if((planets[ii].color <1)||(planets[ii].color >3))
							planets[ii].color = rand() % 3 + 1;
		if(planets[ii].mass > 100) planets[ii].mass = 1.0;
		circle(planets[ii].x, planets[ii].y,
			planets[ii].radius, planets[ii].color);
	}
	if(newplanet){
		if((temp_planet.color <1)||(temp_planet.color >3))
							temp_planet.color = rand() % 3 + 1;
		circle(temp_planet.x, temp_planet.y,
			temp_planet.radius, temp_planet.color);
	}
}
void arrow(){
		glBegin(GL_LINES);
		glColor3f(1.0,0.5,0.0);
		glVertex2f(temp_planet.ax,temp_planet.ay);
		glVertex2f(temp_planet.x, temp_planet.y);
		glColor3f(1.0,0.0,1.0);
		glEnd();
}
bool CTZ(const particle p){
	return ((p.mass <= 0.0)||
		/*(p.mass > 999999999999999)||*/
		(p.x*p.x > 100000000000000)||(p.y*p.y > 100000000000000)
		);
}
void cleanup(){
	vector<particle>::iterator del;
	del = remove_if(planets.begin(), planets.end(), CTZ );
	planets.erase(del, planets.end());
}
void move(){
	int size = planets.size();
	double dy,dx,rad,g,force,tempx,tempy;
	for(int ii=0; ii<size; ii++){
		planets[ii].x += planets[ii].ax;
		planets[ii].y += planets[ii].ay;
		for(int jj=0; jj<size; jj++){
			if(ii != jj){
				dx = planets[ii].x - planets[jj].x;
				dy = planets[ii].y - planets[jj].y;
				rad = dx*dx + dy*dy;
				if(colision){
					if(rad <=
						(planets[ii].radius*planets[ii].radius +
						planets[jj].radius*planets[jj].radius)){
						if(sticky){
							tempy = planets[jj].ay;
							tempx = planets[jj].ax;
							force =	planets[jj].mass * planets[jj].ax;
							planets[jj].ax -= force / planets[ii].mass;
							// fii = mjj * ajj
							force =	planets[jj].mass * planets[jj].ay;
							planets[jj].ay -= force / planets[ii].mass;
							// aii = fii / mii
							force = planets[ii].mass * tempx;
							planets[ii].ax -= force / planets[jj].mass;
							// law of inertia...
							force = planets[ii].mass * tempy;
							planets[ii].ay -= force / planets[jj].mass;
							// f = m*a ... a = f/m ... m = f / a ...
							}
						else{
							tempy = planets[ii].ay;
							tempx = planets[ii].ax;
							force =	planets[jj].mass * planets[jj].ax;
							planets[ii].ax += force / planets[ii].mass;
							// fii = mjj * ajj
							force =	planets[jj].mass * planets[jj].ay;
							planets[ii].ay += force / planets[ii].mass;
							// aii = fii / mii
							force = planets[ii].mass * tempx;
							planets[jj].ax += force / planets[jj].mass;
							// law of inertia...
							force = planets[ii].mass * tempy;
							planets[jj].ay += force / planets[jj].mass;
							// f = m*a ... a = f/m ... m = f / a ...
						}
						if(absorb){
							if(planets[ii].mass < planets[jj].mass){
								planets[ii].color = planets[jj].color;
								planets[ii].x = planets[jj].x;
								planets[ii].y = planets[jj].y;
							}
							planets[ii].mass += planets[jj].mass;
							planets[ii].radius = planets[ii].mass * 2;
							planets[ii].ax = planets[ii].ax / planets[ii].mass;
							planets[ii].ay = planets[ii].ay / planets[ii].mass;
								planets[jj].ax = planets[jj].ay = planets[jj].x = planets[jj].y
									= planets[jj].radius = planets[jj].mass = 0.0;
								cleanup();
						}
					}
				}
				g = G * planets[jj].mass / rad;
				rad = sqrt(rad);
				planets[ii].ax += g * (dx/rad);
				planets[ii].ay += g * (dy/rad);
			}
		}
		// */
	}
}
void draw_lines(){
	double Fg, iFg, rad, dx,dy;
	int size = planets.size();
	glBegin(GL_LINES);
	for(int ii=0; ii < size; ii++){
		for(int jj=0; jj<size; jj++){
			dx = planets[ii].x - planets[jj].x;
			dy = planets[ii].y - planets[jj].y;
			rad = dx*dx + dy*dy;
			Fg = (G * planets[jj].mass/ rad) * planets[ii].mass;
			iFg = 1 - Fg;
			glColor3f(Fg,planets[ii].ax,planets[ii].ay);
			glVertex2f(planets[ii].x, planets[ii].y);
			glColor3f(iFg,planets[jj].ax,planets[jj].ay);
			glVertex2f(planets[jj].x, planets[jj].y);
		}
	}
	glEnd();
}
/***************************/
/* General Glut Functions: */
void display(void) {
	width = glutGet(GLUT_WINDOW_WIDTH) /2;
	height = glutGet(GLUT_WINDOW_HEIGHT) /2;
	glClear(GL_COLOR_BUFFER_BIT);
	glPushMatrix();
	glTranslated((width),(height),0);
	glScaled(scale,scale,1);    /* Zoom */
	glTranslated((-T_X),(-T_Y),0); /* move */
	/* Draw Here */
	if(newplanet) arrow();
	if(line) draw_lines();
	draw_planets();
	glPopMatrix();
	glutSwapBuffers();
}
void init (void) {
	/*General Init: */
  glClearColor(0.0, 0.0, 0.0, 0.0);
  glMatrixMode(GL_PROJECTION);
  glLoadIdentity();
  gluOrtho2D(0,400,400,0);
  glMatrixMode(GL_MODELVIEW);
 /* Call specific Init: */
  glEnable(GL_LINE_SMOOTH);
	 glEnable(GL_POINT_SMOOTH);
}
void add_fountain(){
	particle p;
	p.x = 0.0001 * rand();
	p.radius = p.mass = 1 + 0.0001 * rand();
	p.y = 0.0001 * rand();
	p.ax = p.ay = 0.0001*G*G;
	planets.insert(planets.begin(), p);
 glutPostRedisplay();
}
void add_stream(){
	particle p;
	p.x = -90 + (rand() % 90);
	p.radius = 2.0;
		p.mass = 1.0 + ( (rand() % 100)/100);
	p.y = -90;
	p.ax = p.ay = 2.5;
	p.color = 3;
	planets.insert(planets.begin(), p);
 glutPostRedisplay();
}
void idle (void){
	/* General: */
	if(!pause){
		move();
		cleanup();
		glutPostRedisplay();
	}
	if(refresh && pause){
		glutPostRedisplay();
	}
	if(fountain && !pause) add_fountain();
	if(stream && !pause)  add_stream();
}
void keyboard(unsigned char dakey, int x, int y) {
	switch (dakey) {
	 /***************************/
		/* Zoom and View Controls: */
		case '=':
			if(newplanet){
				temp_planet.mass += temp_planet.mass/2;
				temp_planet.radius = temp_planet.mass*2;
				glutPostRedisplay();
			}
			break;
		case '-':
			if(newplanet){
				temp_planet.mass -= temp_planet.mass/2;
				temp_planet.radius = temp_planet.mass*2;
				glutPostRedisplay();
			}
			break;
	 case 'r':
		 scale = 1.0;
		 T_Y = T_X = 0;
		 break;
	 case 'i':
		 refresh = !refresh; T_Y -= 25 / scale; glutPostRedisplay(); break;
	 case 'k':
		 refresh = !refresh; T_Y += 25 / scale; glutPostRedisplay(); break;
  case 'l':
		 refresh = !refresh; T_X += 25 / scale; glutPostRedisplay(); break;
	 case 'j':
		 refresh = !refresh; T_X -= 25 / scale; glutPostRedisplay(); break;
	 case 'x': case '.' :
		 refresh = !refresh;
		 scale += scale/2; break;
	 case 'z': case ',' :
		 refresh = !refresh;
		 if(scale > 0.0)scale -= scale/2; break;
	 case 'p':
		 pause = !pause;
		 break;
	 case 'q': case 'Q': exit(0); break;
  default: break;
  }
}
void menu(int value) {
	switch(value) {
	 case CNONE:
		 colision = sticky = absorb = 0;
		 break;
	 case STICKY:
		 colision = sticky = 1;
		 absorb = 0;
		 break;
	 case ELASTIC:
		 colision = 1; sticky = absorb = 0;
		 break;
	 case ABSORB:
		 colision = absorb = 1;
		 sticky = 0;
		 break;
	 case FNONE:
		 line = space = 0;
		 break;
	 case LINES:
		 line = !line;
		 break;
	 case CLEAR:
		 planets.erase(planets.begin(), planets.end());
		 glutPostRedisplay();
		 break;
	 case PAUSE:
		 pause = !pause;
		 break;
	 case ORBIT:
		 particle aa, bb, cc;
		 aa.x = aa.y = aa.ax = aa.ay = 0;
		 aa.radius = 10.0; aa.mass = 100.0; aa.color = 1;
		 bb.x = 150; bb.y = bb.ax = 0; bb.ay = 2.6;
		 bb.radius = 1.0; bb.mass = 0.01;
		 cc.x = -30; cc.y = 30; cc.ax = -3.6; cc.ay = -3.6;
		 cc.radius = 1.5; cc.mass = 0.03;
		 planets.erase(planets.begin(), planets.end());
		 planets.insert(planets.begin(), aa);
		 planets.insert(planets.begin(), bb);
		 planets.insert(planets.begin(), cc);
		 bb.x = 0; bb.y = 85; bb.ax = -3.6; bb.ay = 0;
 		 planets.insert(planets.begin(), bb);
		 cc.x = cc.y = 175; cc.ax = 1.9; cc.ay = 0;
		 cc.mass = 0.003; cc.radius = 1.6;
		 planets.insert(planets.begin(), cc);
		 cc.x = cc.y = -300; cc.ax = 1.6; cc.ay = 0;
		 cc.mass = 0.003; cc.radius = 1.6;
		 planets.insert(planets.begin(), cc);
		 cc.x = cc.y = -600; cc.ax = 0.15; cc.ay = 0;
		 cc.mass = 0.00003; cc.radius = 1.06;
		 planets.insert(planets.begin(), cc);
		 cc.x = cc.y = -700; cc.ax = 0.2; cc.ay = 0;
		 cc.mass = 0.00003; cc.radius = 1.006;
		 planets.insert(planets.begin(), cc);
		 glutPostRedisplay();
		 break;
	 case RANDOM:
	 	 particle dd;
		 int xx, ee;
		 ee = rand() % 50;
		 planets.erase(planets.begin(), planets.end());
		 for(xx = 0; xx < ee; xx++){
			 dd.x = rand() % 150; dd.y = rand() % 150;
			 dd.mass = dd.radius = rand() % 25;
			 dd.ax = rand() % 2; dd.y = rand() % 2;
			 planets.insert(planets.begin(), dd);
		 }
		 glutPostRedisplay();
		 break;
	 case EXPLODE:
		 explode = !explode;
		 break;
	 case FOUNT:
		 fountain = !fountain;
		 break;
 	 case STREAM:
		 stream = !stream;
		 break;
	 case ZOOM:
		 scale = 1.0;
		 glutPostRedisplay();
		 break;
	 case ZOOMUP:
		 scale += scale/2;
		 glutPostRedisplay();
		 break;
	 case ZOOMDOWN:
		 if(scale > 0.0) scale -= scale/2;
 		 glutPostRedisplay();
		 break;
	 case CENTER:
		 T_X = T_Y = 0;
		 glutPostRedisplay();
		 break;
	 case MASSI:
		if(newplanet){
			temp_planet.mass += temp_planet.mass/2;
			temp_planet.radius = temp_planet.mass/2;
			glutPostRedisplay();
		}
		 break;
	 case MASSD:
		if(newplanet){
			temp_planet.mass -= temp_planet.mass/2;
			temp_planet.radius = temp_planet.mass/2;
			glutPostRedisplay();
		}
		 break;
  case QUIT: exit(0); break;
  }
}
void MakeMenu(){
	// create menu here...
	int COLIDE, FORCED, PREDEF, MZ;
	 MZ = glutCreateMenu(menu);
	 glutAddMenuEntry("No Zoom ", ZOOM);
	 glutAddMenuEntry("Zoom++ <+>", ZOOMUP);
	 glutAddMenuEntry("Zoom-- <->", ZOOMDOWN);
	 glutAddMenuEntry("center <r> move:(i,j,k,l)", CENTER);
	 COLIDE = glutCreateMenu(menu);
	 glutAddMenuEntry("none" , CNONE);
	 glutAddMenuEntry("elastic" , ELASTIC);
	 glutAddMenuEntry("absorbitive" , ABSORB);
	 glutAddMenuEntry(" \"sticky\" ", STICKY);
	 FORCED = glutCreateMenu(menu);
	 glutAddMenuEntry("none" , FNONE);
	 glutAddMenuEntry("force - lines" , LINES);
	 // glutAddMenuEntry("color - space" , SPACE);
	 PREDEF = glutCreateMenu(menu);
	 glutAddMenuEntry("Orbit", ORBIT);
	 glutAddMenuEntry("Random", RANDOM);
	 glutAddMenuEntry("Particle Fountain", FOUNT);
	 glutAddMenuEntry("Particle Stream", STREAM);
	 glutAddMenuEntry("explode", EXPLODE);
	 glutCreateMenu(menu);
	 glutAddSubMenu("Zoom (z/x)",MZ);
	 glutAddSubMenu("Collision", COLIDE);
	 glutAddSubMenu("Force Display", FORCED);
	 glutAddSubMenu("Predefined", PREDEF);
 	 glutAddMenuEntry("Mass ++", MASSI);
	 glutAddMenuEntry("Mass --", MASSD);
  glutAddMenuEntry("Clear", CLEAR);
	 glutAddMenuEntry("Pause <p>", PAUSE);
	 glutAddMenuEntry("================", NADA);
	 glutAddMenuEntry("Point and click ", NADA);
	 glutAddMenuEntry(" to add planets ", NADA);
	 glutAddMenuEntry("second click for", NADA);
	 glutAddMenuEntry("velocity vector ", NADA);
	 glutAddMenuEntry("================", NADA);
	 glutAddMenuEntry("<<< Quit >>>", QUIT);
  glutAttachMenu(GLUT_RIGHT_BUTTON);
}
/*****************************************************************\
  Mouse functions!   Transforms Mouse Points to relative GL
	            coordinates... by far the hardest part!
\**********************/
void mouse(int button, int state, int Mx, int My) {
	switch(button){
		case GLUT_LEFT_BUTTON:
			if (state == GLUT_DOWN){
				if(!newplanet && !explode){
					newplanet = !newplanet;
					if(set) set = !set;
					temp_planet.x = ((Mx - width) / scale) + T_X;
					temp_planet.y = ((My - height) / scale) + T_Y;
					temp_planet.mass = 2.5;
					temp_planet.radius = temp_planet.mass*2;
					temp_planet.color = (rand() % 3) + 1;
				}
				else if(!set && ! explode){
					if ((((Mx - width) / scale) + T_X) == temp_planet.ax )
						set = !set;
					temp_planet.ax = ((Mx - width) / scale) + T_X;
					temp_planet.ay = ((My - height) / scale) + T_Y;
				}
				else if (!explode){
					temp_planet.ax = ((Mx - width) / scale) + T_X;
					temp_planet.ay = ((My - height) / scale) + T_Y;
					temp_planet.ax -= temp_planet.x;
					temp_planet.ay -= temp_planet.y;
					temp_planet.ax /= 10;
					temp_planet.ay /= 10;
					newplanet = !newplanet;
					if(set) set = !set;
					planets.insert(planets.begin(), temp_planet);
				}
				if(explode){
					 particle p;
					 int xx, ee;
					 ee = rand() % 10;
					 for(xx = 0; xx < ee; xx++){
								p.x = ( ( (Mx - width)/(scale) ) + T_X) + (50 - (rand() % 100));
								p.radius =
									p.mass =
									1.0 + ( (rand() % 30)/10);
								p.y = ( ( (My - width)/(scale) ) + T_X) + (50 - (rand() % 100));
								p.ax = 1.00 - (rand() % 200)/100;
								p.ay = 1.00 - (rand() % 200)/100;
								p.color = (rand() % 3)+1;
								planets.insert(planets.begin(), p);
					 }
				}
				glutPostRedisplay();
			} /* if */
			break;
	}
}
void MouseMove(int newx, int newy){
	if(newplanet){
		temp_planet.ax = ( ( (newx - width)/(scale) ) + T_X);
		temp_planet.ay = ( ( (newy - height)/(scale) ) + T_Y);
		if(!set) set = !set;
		glutPostRedisplay();
	}
}
int main(int argc, char** argv) {
	 glutInit(&argc, argv);
  glutInitDisplayMode (GLUT_DOUBLE | GLUT_RGB);
  glutInitWindowSize (2*width, 2*height);
  glutInitWindowPosition (100, 100);
  glutCreateWindow ("Gravity Works! <right-click-for-menu>");
  init();
  draw_circle();
	 glutDisplayFunc(display);
  glutIdleFunc(idle);
  glutKeyboardFunc(keyboard);
	 glutMouseFunc(mouse);
	 glutPassiveMotionFunc(MouseMove);
	 MakeMenu();
	 menu(ORBIT);
  glutMainLoop();
  return(0);  /* ANSI C requires main to return int. */
}
```

