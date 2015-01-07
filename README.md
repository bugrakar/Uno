# Uno
Uno game project
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.FontMetrics;
import java.awt.Graphics;
import java.awt.Graphics2D;
import java.awt.GridBagLayout;
import java.util.ArrayList;
import java.util.Random;

import javax.swing.BorderFactory;
import javax.swing.JButton;
import javax.swing.JPanel;
import javax.swing.border.Border;

public class Uno {
    private ArrayList<Player> players = new ArrayList<Player>();
    private int turnNum;
    private JPanel topCards = new JPanel();
    private Card topCard;
    private int numPlayers;
    public static Uno uno;
    private static GuiUno gui;
    private int direction = 1;
    private int draw4 = 0;
//1 is going clockwise
    
/* 
NOTE ON CARD NUMBERS:
   -1 is a wild,
   -2 is a draw2
   -3 is a skip
   -4 is a reverse
   -5 is a wild draw 4 */
    public Uno(int numPlayers, int initialCards) {
        gui = new GuiUno();
	gui.setVisible(true);
	this.numPlayers = numPlayers;
	
	topCards.setLayout(new GridBagLayout());
	gui.panel.add(topCards,BorderLayout.CENTER);
	for (int i = 0; i < numPlayers; i++) {
	    players.add(new Player(i));

	}
	for (Player p : players) {
            for (int i = 0; i < initialCards; i++) {
                p.drawCard(p.randomCard());
            }
	}
        this.showtheTurn(players.get(0));
        
    }
    
    //makes the turnNum valid
    private void turningNum() {
        if (turnNum>=numPlayers || turnNum < 0) {
            turnNum = turnNum % numPlayers;
        }
        while (turnNum < 0) {
            turnNum += numPlayers;
        }
    }
    
    private void showtheTurn(Player p) {

        Border line = BorderFactory.createLineBorder(Color.yellow, 4);

        p.getGuiHand().setBorder(line);
    }
    public void turn() {
        Random r = new Random();
        int ran;

	// go to the next player: let him pick his card/have ai choose card
        turnNum=turnNum+(1*direction);
        turningNum();
        
        Player player = players.get(turnNum);
	if (draw4==1){
	    draw4=0;
	    player.drawCard(player.randomCard());
            player.drawCard(player.randomCard());
	    player.drawCard(player.randomCard());
            player.drawCard(player.randomCard());
	}
        switch (topCard.getNumber()) {
            case -2:
                player.drawCard(player.randomCard());
                player.drawCard(player.randomCard());
                break;
            case -4:
                direction=direction*-1;
                turnNum=turnNum+(2*direction);
                break;
            case -3:
                turnNum=turnNum+(1*direction);
                break;
            case -1:
                
                turnNum=turnNum-(1*direction);
                break;
            case -5:
		draw4=1;
                turnNum=turnNum-(1*direction);
                break;
        }
        turningNum();
        player = players.get(turnNum);
	    //else if they choose to draw (some sort of input recognition here)
		//players.get(turnNum).drawCard(player.randCard());
	    //otherwise play a card of their choosing


        if (!player.isHandPlayable()){
            player.drawCard(player.randomCard());
            turn();
        }
        
        for (int i = 0; i < numPlayers; i++) {
            players.get(i).getGuiHand().setBorder(BorderFactory.createEmptyBorder());
        }
        this.showtheTurn(players.get(turnNum));
        if (isWon()) {
            gui.panel.removeAll();

            gui.add(new JPanel() {
                @Override
                public void paintComponent(Graphics g) {
                    
                    Graphics2D g2 = (Graphics2D) g;
                    Font f = GuiUno.font.deriveFont((float)20);
                    FontMetrics metrics = g.getFontMetrics(f);
                    g2.setFont(f);
                    g2.setColor(Color.BLACK);
                    g2.drawString("VICTORY", (int)(gui.getWidth() - metrics.stringWidth("VICTORY"))/2, (int)gui.getHeight()/2);
                    gui.revalidate();
                    gui.repaint();
                }
                
            }, BorderLayout.CENTER);

        }
    }


    
    
    public boolean isWon() {
	for (Player p : players) {
	    if (p.getHand().size() == 0) {
		return true;
	    }
	}
	return false;
    }

    public Card getTopCard() {
	return topCard;
    }
    
    public void setTopCard(Card c) {
        topCard = c;
        topCards.removeAll();
        topCards.add(c);
        

        turn();
        
    }
    
    public static GuiUno getGUI() {
        return gui;
    }
    public void addCardToGUI(Card c) {
        gui.add(c);
    }
    
    public ArrayList<Player> getPlayers() {
        return players;
    }
    
    public int getTurnNum() {
        return turnNum;
    }
}
