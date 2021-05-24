---
title: "Java Spirograph"
author: "Christopher Voigtmann"
date: "2021-02-15"
meta_img: images/Spirograph.JPG
---

![Spirograph](/images/Spirograph.JPG)

Ein Spirograph ist ein geometrisches Spielzeug, mit dem man aus mehreren Zahnrädern Muster und Kurven zeichnen kann. Diese Kurven lassen sich auch mathematisch Darstellen, wie im folgenden Code zu sehen ist.




##### Code Spirograph

```Java
import java.awt.*;
import java.lang.Math;
import javax.swing.JComponent;


public class Spirograph extends JComponent // Erweitert JComponent
{
	//Anlegen der genutzten Datentypen in "zeichnender" Klasse
	private double zahn1, zahn2, t; 
	private int rad1, rad2;
	private int [] x, y; 		//Array für Polygon-Zeichnung
	private int steps = 800;    // Schrittweite, bzw. Anzahl der zu errechnenden Werte für Polygon
	private boolean draw;		
	private boolean fill;
	
	
	
	
	public Spirograph (int r1, int r2, double n1, double n2 )
	{
	///////////////Kontruktor für "zeichnende" Klasse
	/////////////// Initalisierung der Objekte, Einfügen der Übergabewerte in anzulegendes Objekt
	
		rad1 = r1;
		zahn1 = n1;
		t = 2*Math.PI;
		rad2 = r2;
		zahn2 = n2;
		
		
		x = new int [steps];
		y = new int [steps];
		
		draw = false;
		fill = true;
		
	}
	
	// Zeichnender Teil
	 @Override
	    public void paintComponent (Graphics g)
	    {
	        Graphics2D g2 = (Graphics2D) g;
	        g2.setRenderingHint (RenderingHints.KEY_ANTIALIASING,
	        RenderingHints.VALUE_ANTIALIAS_ON);
	        
	        //Trennline 
	        g2.setStroke (new BasicStroke (5));
	        g2.setPaint(Color.black);
	        g2.drawLine(300, 0, 300, 1000);
	        
	        
	        //Kreise zur visuellen Dasrtellung der Räder
	        g2.setStroke (new BasicStroke (1));
	        g2.setPaint(Color.blue);
	        
	        g2.drawOval(30, 200, rad2, rad2);
	        g2.fillOval(30, 600, rad1, rad1);
	        
	        // Freigabe zum beschreiben der Werte für Polygon
	        if(draw==true)
	        {
	        	t = 2*Math.PI;
	        	for (int i = 0 ; i < steps; i++) 
	        	{
	        		t -=  (2*Math.PI)/steps;
	        		x [i] = (int) (650 + rad1*Math.cos(zahn1*t)+rad2*Math.cos(zahn2*t));
	        		y [i] = (int) (500 + rad1*Math.sin(zahn1*t)+rad2*Math.sin(zahn2*t));
	        	}
	      
	        	// Wechsel zwischen Polygon mit und ohne Füllung
	        	if(fill == false)
	        	{
	        		g2.fillPolygon(x, y, steps);
	        	}
	        	else
	        	{
	        		g2.drawPolygon(x, y, steps);
	        	}
	        }   
	    }
	 
	 //Set-Methoden, um von Frame-Klasse aus Werte in "zeichnender" Klasse zu ändern
	 void setSizes (int r1, int r2, double n1, double n2 )
	 {
		 //Setzt, für in Frame-Klasse angelegtes "zeichnendes" Objekt, neue Werte für die Räder 
			rad1 = r1;
			zahn1 = n1;
			
			rad2 = r2;
			zahn2 = n2;
	 }
	 
	 void setPermissiontoDraw (boolean set)
	 {
		 draw = set;
	 }
	 
	 void setFillorDraw (boolean set)
	 {
		 fill = set;
	 }
	
}
```


##### Code Spirographplay

```Java
import java.awt.*;
import java.awt.event.*;
import javax.swing.*;


public class Spirographplay extends JFrame implements ActionListener // Erweitert JFrame, führt eine Action aus
{

	//Anlegen der genutzten Datentypen im Frame
    private JButton buttonDraw, buttonClear , buttonFill;
    
    private JPanel up, down;
    
    private JTextField tfOuterRad;
    private JTextField tfOuterZ;
    private JTextField tfInnerRad;
    private JTextField tfInnerZ;
    
    private int count; // counter für Wechsel "Fill" 
    private Spirograph spr; // Datentyp-Klasse, "zeichnendes" Element
	
	public Spirographplay() {
		///////////////Kontruktor für Frame mit allen Komponenten
		/////////////// In Frame 2 Panel, 1xPanel für Textfields und 1x Panel für Buttons
	
		spr = new Spirograph (150,100,5,16); //Initialisieren der "zeichnenden" Klasse
		count = 0;
		
		
		// Anzeige Radien und Zähne Oben
		up = new JPanel();
		up.setLayout(new GridLayout(2,2));
		
        JLabel outerR = new JLabel("    Radius des aeusseren Kreises");
        up.add(outerR);
        
        tfOuterRad = new JTextField("100", 15);
        tfOuterRad.addActionListener(this);
        up.add(tfOuterRad);
        
        JLabel outerZ = new JLabel ("    Anzahl Zaehne des aeusseren Kreise");
        up.add(outerZ);
        
        tfOuterZ = new JTextField("16", 15);
        tfOuterZ.addActionListener(this);
        up.add(tfOuterZ);
        
        
        JLabel innerR = new JLabel ("    Radius des inneren Kreises");
        up.add(innerR);
        
        tfInnerRad = new JTextField("150", 15);
        tfInnerRad.addActionListener(this);
        up.add(tfInnerRad);
        
        JLabel innerZ = new JLabel ("    Anzahl Zaehne des inneren Kreise");
        up.add(innerZ);
        
        tfInnerZ = new JTextField("5", 15);
        tfInnerZ.addActionListener(this);
        up.add(tfInnerZ);
        /////////////////////////////////////////////////////////////////    
        
    
        
        // Buttons (Draw, Clear und Fill Unten)
        down = new JPanel();
        down.setLayout(new GridLayout(1,2));
        
        buttonDraw = new JButton("Draw");
        buttonDraw.addActionListener(this);
      	down.add(buttonDraw);
      	
      	buttonClear = new JButton("Clear");
      	buttonClear.addActionListener(this);
       	down.add(buttonClear);
       	
       	buttonFill = new JButton("Fill");
       	buttonFill.addActionListener(this);
       	down.add(buttonFill);
        ////////////////////////////////////////////////////
		
		// Frame
		add(up,BorderLayout.PAGE_START);
		add(down,BorderLayout.PAGE_END);
		add (spr);
		
		
		setTitle ("Spirograph");  // Titel setzen
	    setSize (1000, 1000);   // Größe: 1000 x 1000 Pixel
	   setVisible (true);    // sichtbar machen
	    ////////////////////////////////////////////////////////

	    
	    // Timer für regelmäßige Ausführung des actionPerformed
		Timer tim = new Timer (100, this);
		tim.start ();
		/////////////////////////////////////////////////////////
		
	}
	
	public static void main(String[] args) {
	
		// Main (ausgeführtes) Programm
		// Erstellen eines Frameobjekts mit all seinen Inhalten
		Spirographplay sp = new Spirographplay ();
		
	}
	
	
	@Override
	public void actionPerformed(ActionEvent arg0) {
		
		
		// Sicherstellung, dass nur bei Betätigung des Buttons eine "Aktion" passiert 
		// Aufruf der Set-Methoden der "zeichnenden" Klasse
		if (arg0.getSource() == this.buttonDraw)
		{
			spr.setPermissiontoDraw(true);
			spr.repaint();
		}
		
		else if (arg0.getSource() == this.buttonClear)
		{
			spr.setPermissiontoDraw(false);
			spr.repaint();
		}
		
		else if (arg0.getSource() == this.buttonFill)
		{
			count++;
			
			if (count % 2 == 0)
			{
				spr.setFillorDraw(true);
			}
			
			else
			{
				spr.setFillorDraw(false);
			}
			
			spr.repaint();
		}
		
	// Fehlervermidung durch "Try-Catch-Funktion"
	// try wird nur ausgeführt, wenn die Werte in den Textfields verarbeitet werden können.
		
		try {
			// Umwandlung der Strings im Textfields in Zahlen-Datentypen 
			int r1 = Integer.parseInt (this.tfInnerRad.getText());
			int r2 = Integer.parseInt (this.tfOuterRad.getText());
			double n1 = Double.parseDouble(this.tfInnerZ.getText());
			double n2 = Double.parseDouble(this.tfOuterZ.getText());
			spr.setSizes(r1, r2, n1, n2);
			spr.repaint();
		    
			} 	catch(Exception e)
				{
					// System-Nachricht (Console) im Fehlerfall
					System.out.println("Eingabe in Textfelder nicht richtig. "
									+ "Felder dürfen nicht leer sein, oder Buchstaben enthalten!");
				}
		
		
		
		
	}

}
```	
	