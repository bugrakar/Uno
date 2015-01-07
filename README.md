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





import javax.swing.*;
import java.awt.*;
import java.awt.geom.*;
import java.awt.Color;
import java.awt.event.*;

public class Card extends JPanel{

    private static Font font;
    private static FontMetrics metrics;
    private int number;
    private Player p;
    /* -1 = a wild,
     -2 = a draw2
     -3 = a skip
     -4 = a reverse */
    private Color color;

    public Card(Player p, int number, Color color) {
        if (font == null) {
            Card.font = GuiUno.font.deriveFont((float) 12);
        }
        this.number = number;
        this.color = color;
        this.p = p;
        final int pos = p.getPos();

	addMouseListener(new MouseAdapter() {
                
                public void mousePressed(MouseEvent e) {
		    if (pos == Uno.uno.getTurnNum()) {
                        playCard();
                    }
		    
		    
		}
	    });
    }
    
    
    @Override
    public Dimension getPreferredSize() {
        return new Dimension(50, 77);
    }


    
  
    private String getText() {
        if (number > 0) {
            return String.valueOf(number);
        } else {
            switch (number) {
                case -1:
                    return "Wild";
                case -2:
                    return "+2 Draw";
                case -3:
                    return "Skip";
                case -4:
                    return "Reverse";
                case -5:
                    return "+4 Wild";
            }
        }
        return String.valueOf(number);
    }

    public String toString() {
        return getText();
    }
    public Color getColor() {
        return color;
    }

    public int getNumber() {
        return number;
    }


    public void setColor(Color color) {
        this.color = color;
    }

   @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        Graphics2D g2 = (Graphics2D) g;
        int x = 0;
        int y = 0;
        if (metrics == null) {
            metrics = g.getFontMetrics(font);
        }
        g2.setRenderingHint(RenderingHints.KEY_TEXT_ANTIALIASING, RenderingHints.VALUE_TEXT_ANTIALIAS_ON);
	if (this.number != -1 && this.number != -5) {
            if(this.color == Color.YELLOW) {
                g2.setColor(new Color(245,202,5));
            } else {
                g2.setColor(this.color);
            }
	} else {
	    g2.setColor(Color.BLACK);
	}
        g2.fillRect(0, 0, 50, 77);
        g2.setColor(Color.WHITE);
        g2.setFont(this.font);
        g2.drawString(getText(), 3, 19);
                  
        g2.rotate(Math.PI, 48, 66);
        g2.drawString(getText(), 48, 66);
    
    }

    public boolean playCard() {
	boolean  v = p.playCard(this);
      	return v;
    }
}






import java.util.*;
import java.awt.*;
import javax.swing.*;

public class Player {
	private ArrayList<Card> hand = new ArrayList<Card>();
	private JPanel guiHand = new JPanel();
	private Color[] colors = { Color.RED, Color.GREEN, Color.YELLOW,
			Color.BLUE, Color.BLACK };

	private final int POS;

	public Player(int i) {
		this.POS = i;
		if (i == 0 || i == 2) {
			guiHand.setLayout(new FlowLayout());
		} else {
			guiHand.setLayout(new BoxLayout(guiHand, BoxLayout.Y_AXIS));
		} 
		switch (i) {
		case 0:
			Uno.getGUI().panel.add(guiHand, BorderLayout.NORTH);
			break;
		case 1:
			Uno.getGUI().panel.add(guiHand, BorderLayout.WEST);
			break;
		case 2:
			Uno.getGUI().panel.add(guiHand, BorderLayout.SOUTH);
			break;
		case 3:
			Uno.getGUI().panel.add(guiHand, BorderLayout.EAST);
			break;
		default:
			Uno.getGUI().panel.add(guiHand, BorderLayout.CENTER);
		}

	}

	public void drawCard(Card c) {
		hand.add(c);
		guiHand.add(c);


	}

	public boolean isCardPlayable(Card c) {
		// Matching number OR matching color means a card can be played.
		// and -1 means a wild card, which can always be played. same for -5
		// (wild draw four)
		if (Uno.uno.getTopCard() == null) {
			return true;
		}
		if (c.getColor() == Uno.uno.getTopCard().getColor()
				|| c.getNumber() == Uno.uno.getTopCard().getNumber()
				|| Uno.uno.getTopCard().getNumber() == -1
				|| Uno.uno.getTopCard().getNumber() == -5
				|| c.getNumber() == -1 || c.getNumber() == -5) {
			return true;
		} else {
			return false;
		}
	}

	public boolean isHandPlayable() {
		for (int i = 0; i < hand.size(); i++) {
			if (isCardPlayable(hand.get(i))) {
				return true;
			}
		}
		return false;
	}

	public ArrayList<Card> getHand() {
		return hand;
	}

	public boolean playCard(Card c) {

		if (isCardPlayable(c)) {
			hand.remove(c);
			guiHand.remove(c);

			Uno.uno.setTopCard(c);

			return true;
		}
		return false;
	}

	public Card randomCard() {
		Random r = new Random();
		int num;
		num = r.nextInt(15) - 5;
		int ran = r.nextInt(4);
		Card c = new Card(this, num, colors[ran]);
		return c;
	}

	public int getPos() {
		return POS;
	}

	public JPanel getGuiHand() {
		return guiHand;
	}
}





import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.FlowLayout;
import java.awt.Font;
import java.awt.Graphics;
import java.awt.GridLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;
import javax.swing.JButton;
import javax.swing.JComboBox;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;
import javax.swing.JPanel;
import javax.swing.JTextField;

public class GuiOptions {
	public static void main(String[] args) {

		JFrame frame = new JFrame();
		frame.setTitle("Uno Game Options");
		frame.setSize(300, 200);
		
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

		JPanel panel = new JPanel();
		panel = (JPanel) frame.getContentPane();
		panel.setLayout(new GridLayout(4, 0));
		
		
		JComboBox playerChoices = new JComboBox(new String[] { "2", "3", "4" });

		JButton start = new JButton("Start");
		JButton help = new JButton("Help");

		JLabel cardsLabel = new JLabel("Number of Cards");
		JLabel playerLabel = new JLabel("Number of Players");

		JTextField cardField = new JTextField("6" ,1);
	
		

		JPanel numCards = new JPanel();
		JPanel numPlayers = new JPanel();

		numCards.add(cardsLabel);
		numCards.add(cardField);
		numCards.setLayout(new FlowLayout());

		numPlayers.add(playerLabel);
		numPlayers.add(playerChoices);
		numPlayers.setLayout(new FlowLayout());

		start.addActionListener(new buttonhandler(frame, playerChoices,
				cardField));
		help.addActionListener(new buttonhandler(frame, playerChoices,
				cardField));
		panel.add(numCards);
		panel.add(numPlayers);
		panel.add(start);
		panel.add(help);
		frame.setVisible(true);
	}

}

class buttonhandler implements ActionListener {
	JFrame f;
	JComboBox box;
	JTextField textfield;

	public buttonhandler(JFrame f, JComboBox box, JTextField textfield) {
		this.f = f;
		this.box = box;
		this.textfield = textfield;
	}

	public void actionPerformed(ActionEvent e) {
		JButton b = (JButton) e.getSource();

		if (b.getText().equals("Start")) {
			int v;
			try {
				v = Integer.parseInt(textfield.getText());
			} catch (Exception ee) {
				v = 6;
			}
			Uno game = new Uno(
					Integer.parseInt((String) box.getSelectedItem()), v);
			Uno.uno = game;
			f.setVisible(false);

			
		}

		if (b.getText().equals("Help")) {

			JOptionPane.showMessageDialog(null, "Playing the Game \n " +

					"- Naturally, the object of Uno is to get rid of all your cards before anyone else does. \n" +					
					"- Each player has to match the card in the discard pile either by number, color or word. \n"
					+ "For example, if the top card on the discard pile is a red 7, a player must throw down a red card or any color 7. Or the player can throw down a Wild Card. \n "
					+ "If the player doesn't have anything to match, he must pick a card from the draw pile. \n" +
					"If he can play what is drawn, great. Otherwise play moves to the next person. You do not have to continue picking up until you come across a card you can play. \n"+
					"- So what's a Wild? This type of card lets you change the color of play."
					+ "- For example, if there is a blue 3 on top of the pile, and you have more yellow cards you want to play, \n"
					+ " You could lay down a wild card."
					+ "- You can change the color to yellow by laying down a yellow card, and the next player must play cards of that color.\n"
					+ " Or change the color again with another wild. \n"+			
					"- You'll also come across cards labeled Skip. Quite simply, this card will skip the person next to you, and let the following player have her turn right away. \n"+
					"- Found a Reverse card? If you play that card, then the order of play is reversed. Instead of the player on your left playing after you, it will pass to the one on your right \n"
					+"- Cards labeled Draw 2 or Plus 2 are pickup cards. The player next to you will have to pick up two cards if you play one of these.\n "
					+"- Lastly, you will come across cards called Wild Draw 4. This means that, if played, the player gets to change the color and the player next to him has to pick up four cards. \n"+							
					 "- Once a player has no cards left, the hand is over. Points are scored and you start over again \n"
					);

		}
	}

}

class GuiUno extends JFrame {
	
    public static Font font;
    JPanel panel;
   
    public GuiUno() {
        try {
        	
            this.font = Font.createFont(Font.PLAIN, new File("Action_Man_Bold.ttf"));
        } catch(IOException e) {
            System.out.println("ERROR !!!");
            System.exit(0);
        } catch (Exception e) {
            System.out.println(e);
            System.exit(0);
        }
	this.setTitle("Uno");
	this.setSize(500,700);
	this.setDefaultCloseOperation(EXIT_ON_CLOSE);
        panel = (JPanel) this.getContentPane();
        panel.setLayout(new BorderLayout());
       
        
    }
}
