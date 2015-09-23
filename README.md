//Rishabh Mahajani
//Pd 4
// Program Statement: This program takes an image and then applies the vignette filter to it

package Mahajani;
import java.awt.Canvas;
import java.awt.Color;
import java.awt.Frame;
import java.awt.Graphics;
import java.awt.event.WindowAdapter;
import java.awt.event.WindowEvent;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import javax.imageio.ImageIO;


public class InstagramFilter extends Canvas {
BufferedImage myImage, vignetteImage;
public double largestDistance,averageBright;
public double centerX, centerY;
private static final double WARMNESS_FACTOR = 0.75;
private static final double DARKNESS_FACTOR = 0.47;
private static final double BRIGHTNESS_FACTOR = 0.4;

/**
 * Constructor for the InstagramFilter class
 */
public InstagramFilter() {
	super();
	try{ 
		myImage = ImageIO.read(new File("whale.jpg"));
	} 
	catch(IOException e) { 
		System.out.println("image not found");
		System.exit(0);
	} 
	
	vignetteImage = new BufferedImage(myImage.getWidth(),myImage.getHeight(),myImage.getType());
	vignetteImage.getGraphics().drawImage(myImage,0,0,null); 
	centerX = vignetteImage.getWidth()/2;
	centerY = vignetteImage.getHeight()/2; 
		for (int i=0;i<vignetteImage.getWidth();i++) { 
		for (int j=0;j<vignetteImage.getHeight();j++) { 
			int color = vignetteImage.getRGB(i,j);
			
			int red = (color & 0x00ff0000)>>>16;
			int green = (color & 0x0000ff00)>>>8;
			int blue = color & 0x000000ff;  
			red += red/4 * WARMNESS_FACTOR;
			blue-=blue/4 * WARMNESS_FACTOR;
			green-=green/15 * WARMNESS_FACTOR;
		if(red>255) red=255;
		if(blue<0) blue = 0;
		if(green<0) green = 0;
			
		String col = "0x00ff00ff";
		int newCol = Integer.parseInt(col);
			int warmColor = red<<16|green<<8|blue;
			Color c = new Color(newCol);
			vignetteImage.setRGB(i,j,warmColor);
			
		} 
	} 
		makeBrighter();
		darkenImage();
	
} 

/**\
 * Makes the image brighter before it is vignetted
 */
public void makeBrighter() { 
	float[] myHSBVals = new float[3];
	averageBright = findAverageBrightness();
	
	for (int i=0;i<vignetteImage.getWidth();i++) {
		for (int j=0;j<vignetteImage.getHeight();j++) { 
			int newColor = vignetteImage.getRGB(i, j);
			Color myColor = new Color(newColor);
			int red = (newColor & 0x00ff0000)>>>16;
			int green = (newColor & 0x0000ff00)>>>8;
			int blue = newColor & 0x000000ff;
			myHSBVals = Color.RGBtoHSB(red, green, blue, myHSBVals);
			float brightness = myHSBVals[2];
		if(myHSBVals[2]<0.2*averageBright)
		brightness = (float) (myHSBVals[2] + 0.4*myHSBVals[2]);
		else if(myHSBVals[2]>0.2*averageBright && myHSBVals[2]<0.7*averageBright) {
			brightness = (float) (myHSBVals[2] + 0.3*myHSBVals[2]);
		}
		if(brightness>1) brightness=1;
		else if(brightness<0) brightness=0;
		int hsbColor = Color.HSBtoRGB(myHSBVals[0], myHSBVals[1], brightness);
		vignetteImage.setRGB(i,j,hsbColor);
		
		} 
	} 
} 

/**
 * Vignettes the image by making the image darker as the distance from the xenter increases
 */
public void darkenImage() {
	double distance = 0;
	int count=0;
	for(int i=0;i<vignetteImage.getWidth();i++) {
		for(int j=0;j<vignetteImage.getHeight();j++) { 
			int originalColor = vignetteImage.getRGB(i,j);
			int red = (originalColor & 0x00ff0000)>>>16;
			int green = (originalColor & 0x0000ff00)>>>8;
			int blue = originalColor & 0x000000ff;
			distance = Math.sqrt(Math.pow(centerX-i,2)+Math.pow(centerY-j,2));
			
			if (count==0) { 
				largestDistance = distance;
				count++;
			} 
			
			
			red+= (1-(distance/(largestDistance))) * (BRIGHTNESS_FACTOR * red);
			blue+= (1-(distance/(largestDistance))) * (BRIGHTNESS_FACTOR * blue);
			green+= (1-(distance/(largestDistance))) * (BRIGHTNESS_FACTOR * green);
			
			
			if(red>255) red=255;
			if(blue>255) blue = 255;
			if(green>255) green = 255;
			
			red-= (distance/largestDistance) * (DARKNESS_FACTOR*red);
			blue-= (distance/largestDistance) * (DARKNESS_FACTOR*blue);
			green-= (distance/largestDistance) * (DARKNESS_FACTOR*green);
			
			
			int darkColor = red<<16|green<<8|blue;
			
			vignetteImage.setRGB(i,j,darkColor);
		}
	} 
} 

/**
 * This method finds the average brightness in the image
 * @return the average amount of brightness in the image
 */
public int findAverageBrightness() {
	float[] HSBVals = new float[3];
	int averageBrightness = 0;
	for (int i=0;i<myImage.getWidth();i++) {
		for (int j=0;j<myImage.getHeight();j++) { 
			int newColor = myImage.getRGB(i, j);
			Color myColor = new Color(newColor);
			int red = (newColor & 0x00ff0000)>>>16;
			int green = (newColor & 0x0000ff00)>>>8;
			int blue = newColor & 0x000000ff;
			HSBVals = Color.RGBtoHSB(red, green, blue, HSBVals);
			averageBrightness+=HSBVals[2];
			
		} 
		
	} 
	
	return averageBrightness/(myImage.getWidth() * myImage.getHeight());
	
} 

/**
 * Paints the original image and the vignetted image
 */
public void paint(Graphics g) {
	g.drawImage(myImage,600,0,600,600,null);
	
	g.drawImage(vignetteImage,0,0,600,600,null);
	
} 

/**
 * The main method creates a frame and then makes a WindowAdapter Object to close the frame
 * @param args
 */
public static void main(String[] args) {
	Frame myFrame = new Frame();
	myFrame.add(new InstagramFilter());
	myFrame.setSize(600,600);
	WindowAdapter d = new WindowAdapter() 
	{ 
		public void windowClosing(WindowEvent e) { 
			System.exit(0);
		} 
	}; 
	myFrame.addWindowListener(d);
	myFrame.setVisible(true); 
	
	} 
} 
