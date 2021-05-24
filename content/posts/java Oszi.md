---
title: "Java Signalgenerator"
author: "Christopher Voigtmann"
date: "2020-03-05"
meta_img: images/javaoszi.JPG
---


![javaoszi](/images/javaoszi.JPG)




Das Projekt für die Darstellung einer Oszilloskope Frontblende bestand aus Fünf Teilaufgaben. Mein Teil bestand aus der Verarbeitung und Visualisierung der Werte eines vorgegebenen "Data-Buffers", bwz. eines Signalgenerators.
Interessant war dabei die Umsetzung des Ringspeichers in den Graphen und die Wertweitergabe im Array.




##### Code DataBuffer

```Java
public class DataBuffer
{
    private double [] data;
    private int size, index;
    private long count;
    
    public DataBuffer (int n)
    {
        data = new double [n];
        size = n;
        index = 0;
        count = 0;
    }
    
    public void input (double value)
    {       
        index = (index + 1) % size;
        data [index] = value;
        count++;
    }
    
    public double getValue (int i)
    {
        if (i < size)
        {
            int k = (index - i + size) % size;
            return data [k];
        }
        
        return 0.0;
    }
    
    public long getCount () { return count; }
}
```


##### Code DataView

```Javae
import javax.swing.*;
import java.awt.*;



//Objekt der Klasse Databuffer
public class DataView extends JComponent {

private DataBuffer data;
private int steps;
private double deltat;
private double ymin, ymax;
private int mode;
private int [] x, y;


public DataView (DataBuffer db, int stps, double dt, double ymi, double yma, int mod)
{
setPreferredSize (new Dimension (850, 500));
data = db; //DataBuffer, der angezeigt wird
steps = stps; //Anzahl der Werte, die angezeigt werden - 1. Sollte kleiner als der size-Wert des DataBuffers sein.
deltat = dt; //Zeitdifferenz von einem Wert zum nächsten
ymin = ymi; //Minimaler und maximaler darstellbarer Wert
ymax = yma;
mode = mod; //Anzeigemode (0 oder 1)
x = new int [steps];
y = new int [steps];


}


public void paintComponent (Graphics g)
{

Graphics2D g2 = (Graphics2D) g; //Grafik-Kontext holen

//Rechteck (Rahmen) fuer Werte definieren
g2.setPaint (Color.black);
g2.setStroke (new BasicStroke (2));
int h = (int)ymax + 1 - (int)ymin +1;
int b = (int)steps+1;
int x0 = 100;
int y0 = 100;
g2.drawRect ( x0,y0, b, h);
g2.setStroke (new BasicStroke (2));
g2.drawLine (x0, (int) (ymax) + y0, x0 + b,(int)(ymax) + y0);
String str;

str = String.valueOf(steps+1);
g2.drawString(str, x0-20, y0+h+20);

str = String.valueOf(0);
g2.drawString(str,b+x0-10, y0+h+20);

str = String.valueOf(ymin);
g2.drawString(str,b+x0+10, y0+h-5);

str = String.valueOf(ymax);
g2.drawString(str,b+x0+10, y0+5);




if(mode==0)
{
// Werte mittels Polyline als Graf zeichnen
// Koordinaten in Feld schreiben:


	for (int i = 0; i < steps; i++)
	{
	


	y [i] = (int) (ymax+y0 - h/2 *data.getValue(i) );
	x [i] = x0+b -(((int)deltat+i));
	}



g2.setPaint (Color.blue);
g2.setStroke (new BasicStroke (1));
g2.drawPolyline (x, y, steps); // Funktion zeichnen

}
if (mode==1)
{
	int s = (int) ((data.getCount () % steps));
	
	  
	for (int j = 0; j < s; j++)
	{
		
	
		
		 
	y [j] = (int) (y0+ymax+h/2*data.getValue((int) (+(data.getCount () % steps))-j) );
	
	x [j] = x0 +(((int)j));
	
	

	}
	g2.setPaint (Color.blue);
	g2.setStroke (new BasicStroke (1));
	g2.drawPolyline (x, y, s);
	
	
	for (int j=s;j<steps;j++)
	{
		int k = j-s;
		y [k] = (int) (y0+ymax+h/2*data.getValue((int) (s+steps-j)));
		
		x [k] = x0 +(((int)deltat+j));
		
	}
	



g2.setPaint (Color.red);
g2.setStroke (new BasicStroke (1));
g2.drawPolyline (x, y, steps-s); // Funktion zeichnen

g2.setPaint(Color.gray);
g2.setStroke (new BasicStroke (3));
g2.drawLine(x0+s, y0, x0+s, (int)(y0+ymax-ymin) );
	
}
}

void setBuffer (DataBuffer db)
{
data = db;
}

void setTRange (int stps, double dt)
{
steps = stps;
deltat = dt;
}

void setYRange (double ymi, double yma)
{
ymin = ymi;
ymax = yma;

}

void setMode (int mod)
{
mode = mod;
}

}

```

##### Code DataViewTest

```Java
import java.awt.event.*;
import javax.swing.*;
import javax.swing.Timer;



public class DataViewTest extends JFrame implements ActionListener {

	
	private int size = 250;
	private DataView view;
	private DataBuffer buff;
	public DataViewTest ()
	{
		buff = new DataBuffer(size);
		// Übergabewerte für DataBuffer anpassen
		view = new DataView(buff,size-1,2,-60,60,0);
		add(view);
		
		Timer tim = new Timer (100, this);
		tim.start ();
		
		
		
		 setTitle ("DataViewTest");  // Titel setzen
	      setSize (400, 200);   // Größe: 400 x 200 Pixel
	      setVisible (true);    // sichtbar machen
	      setDefaultCloseOperation 
	      (JFrame.EXIT_ON_CLOSE);
	}
	
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		DataViewTest dvt = new DataViewTest();
		DataBuffer db = new DataBuffer (300);
	
	//	dvt.view.setTRange(800,20); OK
	//	dvt.view.setYRange(-80, +80); OK
	//	dvt.view.setMode(1);OK
		
		//System.out.println(ts.zahl);
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		// TODO Auto-generated method stub
		buff.input((double)((Math.random()*2-1)));
	
		view.repaint();
		
		
		
	}

}

```
