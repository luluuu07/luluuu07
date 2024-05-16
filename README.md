package hey;

import java.io.Serializable;
import javax.swing.JOptionPane;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.image.BufferedImage;
import java.io.*;
import java.util.ArrayList;

public class HotelBookingSystem extends JFrame implements Serializable {
    private Hotel hotel;
    private JTextArea outputArea;
    private String currentUser;
    private JPanel cards;
    private static final String LOGIN_PANEL = "Login Panel";
    private static final String HOTEL_PANEL = "Hotel Panel";
    private BufferedImage backgroundImage;

    public HotelBookingSystem() {
        super("Hotel Booking System");
        this.hotel = new Hotel(10); // Initialize a hotel with 10 rooms


        // Set frame properties
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(800, 600);
        setLocationRelativeTo(null); // Center the frame

        // Initialize components
        setupButtons();
        outputArea = new JTextArea(15, 30);
        outputArea.setEditable(false);

        // Create main panel
        cards = new JPanel(new CardLayout()) {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                if (backgroundImage != null) {
                    g.drawImage(backgroundImage, 0, 0, getWidth(), getHeight(), this);
                }
            }
        };

        // Add login/signup panel
        JPanel loginPanel = createLoginPanel1();
        cards.add(loginPanel, LOGIN_PANEL);

        // Add hotel panel
        JPanel hotelPanel = createHotelPanel();
        cards.add(hotelPanel, HOTEL_PANEL);

        // Add cards panel to frame
        add(cards);

        // Display frame
        setVisible(true);
    }

    private void setupButtons() {
        // Set look and feel to Nimbus for modern appearance
        try {
            UIManager.setLookAndFeel("javax.swing.plaf.nimbus.NimbusLookAndFeel");
            SwingUtilities.updateComponentTreeUI(this);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    

    private JPanel createLoginPanel1() {
        JPanel loginPanel = new JPanel(new BorderLayout());
        loginPanel.setBackground(new Color(252, 252, 252));

        // Title panel
        JPanel titlePanel = new JPanel();
        titlePanel.setBackground(new Color(252, 252, 252));
        JLabel titleLabel = new JLabel("Hotel Booking System");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 30));
        titlePanel.add(titleLabel);
        loginPanel.add(titlePanel, BorderLayout.NORTH);

        // Login details panel
        JPanel loginDetailsPanel = new JPanel(new GridBagLayout());
        loginDetailsPanel.setBackground(new Color(252, 252, 252));
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.gridx = 0;
        gbc.gridy = 0;
        gbc.insets = new Insets(10, 10, 10, 10);

        // User Type radio buttons
        JLabel userTypeLabel = new JLabel("User Type:");
        userTypeLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(userTypeLabel, gbc);

        gbc.gridx++;
        JRadioButton adminRadioButton = new JRadioButton("Admin");
        JRadioButton guestRadioButton = new JRadioButton("Guest");
        adminRadioButton.setFont(new Font("Arial", Font.PLAIN, 16));
        guestRadioButton.setFont(new Font("Arial", Font.PLAIN, 16));
        ButtonGroup userTypeGroup = new ButtonGroup();
        userTypeGroup.add(adminRadioButton);
        userTypeGroup.add(guestRadioButton);

        JPanel userTypePanel = new JPanel();
        userTypePanel.setBackground(new Color(252, 252, 252));
        userTypePanel.add(adminRadioButton);
        userTypePanel.add(guestRadioButton);
        loginDetailsPanel.add(userTypePanel, gbc);

        // Username field
        gbc.gridx = 0;
        gbc.gridy++;
        JLabel usernameLabel = new JLabel("Username:");
        usernameLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(usernameLabel, gbc);

        gbc.gridx++;
        JTextField usernameField = new JTextField(15);
        usernameField.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(usernameField, gbc);

        // Password field
        gbc.gridx = 0;
        gbc.gridy++;
        JLabel passwordLabel = new JLabel("Password:");
        passwordLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(passwordLabel, gbc);

        gbc.gridx++;
        JPasswordField passwordField = new JPasswordField(15);
        passwordField.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(passwordField, gbc);

        // Show Password checkbox
        gbc.gridx = 0;
        gbc.gridy++;
        JCheckBox showPasswordCheckbox = new JCheckBox("Show Password");
        showPasswordCheckbox.setBackground(new Color(252, 252, 252));
        showPasswordCheckbox.setFont(new Font("Arial", Font.PLAIN, 16));
        showPasswordCheckbox.addActionListener(e -> {
            if (showPasswordCheckbox.isSelected()) {
                passwordField.setEchoChar((char) 0);
            } else {
                passwordField.setEchoChar('*');
            }
        });
        loginDetailsPanel.add(showPasswordCheckbox, gbc);

        // Login button
        gbc.gridx = 0;
        gbc.gridy++;
        gbc.gridwidth = 2;
        JButton loginButton = new JButton("Login");
        loginButton.setBackground(new Color(0, 150, 136));
        loginButton.setForeground(Color.WHITE);
        loginButton.setFont(new Font("Arial", Font.BOLD, 18));
        loginButton.addActionListener(e -> {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());
            boolean isAdmin = adminRadioButton.isSelected(); // Check if Admin radio button is selected

            if (authenticate(username, password, isAdmin)) {
                currentUser = username;
                CardLayout cardLayout = (CardLayout) cards.getLayout();
                cardLayout.show(cards, HOTEL_PANEL);
            } else {
                JOptionPane.showMessageDialog(null, "Invalid username or password. Please try again.");
            }
        });
        loginDetailsPanel.add(loginButton, gbc);

        // Signup button
        gbc.gridy++;
        JButton signupButton = new JButton("Sign Up");
        signupButton.setBackground(new Color(0, 172, 193));
        signupButton.setForeground(Color.WHITE);
        signupButton.setFont(new Font("Arial", Font.BOLD, 18));
        signupButton.addActionListener(e -> showSignupDialog(loginPanel, usernameField, passwordField, adminRadioButton, guestRadioButton));
        loginDetailsPanel.add(signupButton, gbc);

        // Add login details panel to login panel
        loginPanel.add(loginDetailsPanel, BorderLayout.CENTER);

        return loginPanel;
    }

    private void showSignupDialog(JPanel loginPanel, JTextField usernameField, JPasswordField passwordField, JRadioButton adminRadioButton, JRadioButton guestRadioButton) {
        JTextField newUsernameField = new JTextField();
        JPasswordField newPasswordField = new JPasswordField();
        JCheckBox showPasswordCheckbox = new JCheckBox("Show Password");
        JRadioButton newAdminRadioButton = new JRadioButton("Admin");
        JRadioButton newGuestRadioButton = new JRadioButton("Guest");

        ButtonGroup group = new ButtonGroup();
        group.add(newAdminRadioButton);
        group.add(newGuestRadioButton);

        JPanel panel = new JPanel(new GridLayout(0, 1));
        panel.setBackground(new Color(252, 252, 252));
        panel.add(new JLabel("New Username:"));
        panel.add(newUsernameField);
        panel.add(new JLabel("New Password:"));
        panel.add(newPasswordField);
        panel.add(showPasswordCheckbox);
        panel.add(newAdminRadioButton);
        panel.add(newGuestRadioButton);

        showPasswordCheckbox.addActionListener(e -> {
            if (showPasswordCheckbox.isSelected()) {
                newPasswordField.setEchoChar((char) 0);
            } else {
                newPasswordField.setEchoChar('*');
            }
        });

        int result = JOptionPane.showConfirmDialog(null, panel, "Sign Up", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);

        if (result == JOptionPane.OK_OPTION) {
            String newUsername = newUsernameField.getText();
            String newPassword = String.valueOf(newPasswordField.getPassword());
            boolean isAdmin = newAdminRadioButton.isSelected(); // Check if admin is selected

            if (newUsername.trim().isEmpty() || newPassword.isEmpty()) {
                JOptionPane.showMessageDialog(null, "Username and password cannot be empty.");
            } else {
                registerUser(newUsername, newPassword, isAdmin); // Call registerUser with isAdmin flag
                JOptionPane.showMessageDialog(null, "Sign up successful. Please login.");

                // Reset login panel fields after sign-up
                usernameField.setText(newUsername);
                passwordField.setText(newPassword);
                if (isAdmin) {
                    adminRadioButton.setSelected(true);
                    guestRadioButton.setSelected(false);
                } else {
                    adminRadioButton.setSelected(false);
                    guestRadioButton.setSelected(true);
                }
            }
        }
    }

    private void registerUser(String username, String password, boolean isAdmin) {
        // Register user based on isAdmin flag
        if (isAdmin) {
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("admin_users.dat"))) {
                ArrayList<User> adminUsers = (ArrayList<User>) ois.readObject();
                adminUsers.add(new User(username, password));
                saveUsers(adminUsers, "admin_users.dat");
            } catch (IOException | ClassNotFoundException e) {
                ArrayList<User> adminUsers = new ArrayList<>();
                adminUsers.add(new User(username, password));
                saveUsers(adminUsers, "admin_users.dat");
            }
        } else {
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("guest_users.dat"))) {
                ArrayList<User> guestUsers = (ArrayList<User>) ois.readObject();
                guestUsers.add(new User(username, password));
                saveUsers(guestUsers, "guest_users.dat");
            } catch (IOException | ClassNotFoundException e) {
                ArrayList<User> guestUsers = new ArrayList<>();
                guestUsers.add(new User(username, password));
                saveUsers(guestUsers, "guest_users.dat");
            }
        }
    }

    private void saveUsers(ArrayList<User> users, String fileName) {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fileName))) {
            oos.writeObject(users);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private boolean authenticate(String username, String password, boolean isAdmin) {
        // Authenticate based on user type
        if (isAdmin) {
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("admin_users.dat"))) {
                ArrayList<User> adminUsers = (ArrayList<User>) ois.readObject();
                for (User user : adminUsers) {
                    if (user.getUsername().equals(username) && user.getPassword().equals(password)) {
                        return true;
                    }
                }
            } catch (IOException | ClassNotFoundException e) {
                e.printStackTrace();
            }
        } else {
            try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("guest_users.dat"))) {
                ArrayList<User> guestUsers = (ArrayList<User>) ois.readObject();
                for (User user : guestUsers) {
                    if (user.getUsername().equals(username) && user.getPassword().equals(password)) {
                        return true;
                    }
                }
            } catch (IOException | ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    private JPanel createHotelPanel() {
        JPanel hotelPanel = new JPanel(new BorderLayout());
        hotelPanel.setBackground(new Color(252, 252, 252));

        // Create button panel
        JPanel buttonPanel = new JPanel(new GridLayout(0, 1, 0, 20));
        buttonPanel.setBackground(new Color(252, 252, 252));
        buttonPanel.add(createStyledButton("Book a Room", new Color(0, 172, 193)));
        buttonPanel.add(createStyledButton("Cancel Booking", new Color(244, 67, 54)));
        buttonPanel.add(createStyledButton("Exit", new Color(158, 158, 158)));

        // Add button panel to hotel panel
        hotelPanel.add(buttonPanel, BorderLayout.NORTH);

        // Create output area for available rooms
        outputArea = new JTextArea(15, 30);
        outputArea.setEditable(false);
        outputArea.setFont(new Font("Arial", Font.BOLD, 18));
        JScrollPane scrollPane = new JScrollPane(outputArea);
        scrollPane.setBorder(BorderFactory.createEmptyBorder());

        // Add output area to hotel panel
        hotelPanel.add(scrollPane, BorderLayout.CENTER);

        // Display available rooms when the hotel panel is created
        displayAvailableRooms();

        return hotelPanel;
    }

    private JButton createStyledButton(String text, Color color) {
        JButton button = new JButton(text);
        button.setFont(new Font("Arial", Font.BOLD, 18));
        button.setForeground(Color.WHITE);
        button.setBackground(color);
        button.setFocusPainted(false);

        button.addMouseListener(new java.awt.event.MouseAdapter() {
            public void mouseEntered(java.awt.event.MouseEvent evt) {
                button.setBackground(color.darker());
            }

            public void mouseExited(java.awt.event.MouseEvent evt) {
                button.setBackground(color);
            }
        });

        if (text.equals("Book a Room")) {
            button.addActionListener(e -> showRoomSelectionDialog());
        } else {
            button.addActionListener(e -> handleButtonClick(text));
        }

        return button;
    }

    private void showRoomSelectionDialog() {
        JPanel panel = new JPanel();
        panel.setLayout(new GridLayout(0, 5, 10, 10));
        panel.setBackground(new Color(252, 252, 252));

        ArrayList<JCheckBox> checkBoxes = new ArrayList<>();

        for (Room room : hotel.getRooms()) {
            if (!room.isBooked()) {
                JCheckBox checkBox = new JCheckBox(Integer.toString(room.getRoomNumber()));
                checkBox.setFont(new Font("Arial", Font.PLAIN, 16));
                checkBox.setBackground(new Color(252, 252, 252));
                panel.add(checkBox);
                checkBoxes.add(checkBox);
            }
        }

        int result = JOptionPane.showConfirmDialog(null, panel, "Select Rooms to Book", JOptionPane.OK_CANCEL_OPTION);
        if (result == JOptionPane.OK_OPTION) {
            for (int i = 0; i < checkBoxes.size(); i++) {
                if (checkBoxes.get(i).isSelected()) {
                    int roomNumber = Integer.parseInt(checkBoxes.get(i).getText());
                    int addPersonChoice = JOptionPane.showConfirmDialog(null, "Do you want to add a person to Room " + roomNumber + "?", "Add Person", JOptionPane.YES_NO_OPTION);
                    if (addPersonChoice == JOptionPane.YES_OPTION) {
                        showAddPersonsDialog(roomNumber);
                    } else {
                        int bookChoice = JOptionPane.showConfirmDialog(null, "Are you sure you want to book Room " + roomNumber + "?", "Book Room", JOptionPane.YES_NO_OPTION);
                        if (bookChoice == JOptionPane.YES_OPTION) {
                            hotel.bookRoom(roomNumber);
                            displayAvailableRooms();
                        } else {
                            JOptionPane.showMessageDialog(null, "Booking for Room " + roomNumber + " cancelled.");
                        }
                    }
                }
            }
        }
    }


    private void showAddPersonsDialog(int roomNumber) {
        Room selectedRoom = null;
        for (Room room : hotel.getRooms()) {
            if (room.getRoomNumber() == roomNumber) {
                selectedRoom = room;
                break;
            }
        }
        if (selectedRoom != null) {
            int maxPersons = selectedRoom.getMaxPersons();
            int currentPersons = selectedRoom.isBooked() ? 1 : 0; // If the room is already booked, there is at least one person
            int remainingCapacity = maxPersons - currentPersons;
            String message = "Enter number of persons (Max " + remainingCapacity + " remaining):";
            JTextField personsField = new JTextField();
            personsField.setPreferredSize(new Dimension(100, 25)); // Set preferred size
            JPanel panel = new JPanel();
            panel.add(new JLabel(message));
            panel.add(personsField);

            int result = JOptionPane.showConfirmDialog(null, panel, "Add Persons to Room " + roomNumber, JOptionPane.OK_CANCEL_OPTION);
            if (result == JOptionPane.OK_OPTION) {
                try {
                    int personsToAdd = Integer.parseInt(personsField.getText());
                    if (personsToAdd > 0 && personsToAdd <= remainingCapacity) {
                        int totalPersons = currentPersons + personsToAdd;
                        selectedRoom.bookRoom(); // Mark the room as booked
                        hotel.addPersonsToRoom(roomNumber, totalPersons);
                        displayAvailableRooms();
                    } else {
                        JOptionPane.showMessageDialog(null, "Invalid input. Please enter a valid number of persons within the remaining capacity.");
                    }
                } catch (NumberFormatException e) {
                    JOptionPane.showMessageDialog(null, "Invalid input. Please enter a valid number.");
                }
            } else {
                JOptionPane.showMessageDialog(null, "Booking for Room " + roomNumber + " cancelled.");
            }
        } else {
            JOptionPane.showMessageDialog(null, "Room " + roomNumber + " does not exist.");
        }
    }



    private JPanel createLoginPanel() {
        JPanel loginPanel = new JPanel(new BorderLayout());
        loginPanel.setBackground(new Color(252, 252, 252));

        // Title panel
        JPanel titlePanel = new JPanel();
        titlePanel.setBackground(new Color(252, 252, 252));
        JLabel titleLabel = new JLabel("Hotel Booking System");
        titleLabel.setFont(new Font("Arial", Font.BOLD, 30));
        titlePanel.add(titleLabel);
        loginPanel.add(titlePanel, BorderLayout.NORTH);

        // User Type selection panel
        JPanel userTypePanel = new JPanel();
        userTypePanel.setBackground(new Color(252, 252, 252));
        JLabel userTypeLabel = new JLabel("User Type:");
        userTypeLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        userTypePanel.add(userTypeLabel);

        // Radio buttons for Admin and Guest
        JRadioButton adminRadioButton = new JRadioButton("Admin");
        JRadioButton guestRadioButton = new JRadioButton("Guest");
        ButtonGroup userTypeGroup = new ButtonGroup();
        userTypeGroup.add(adminRadioButton);
        userTypeGroup.add(guestRadioButton);

        // Add radio buttons to the panel
        userTypePanel.add(adminRadioButton);
        userTypePanel.add(guestRadioButton);

        // Login details panel
        JPanel loginDetailsPanel = new JPanel(new GridBagLayout());
        loginDetailsPanel.setBackground(new Color(252, 252, 252));
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.gridx = 0;
        gbc.gridy = 0;
        gbc.insets = new Insets(5, 5, 5, 5);

        // Username field
        JLabel usernameLabel = new JLabel("Username:");
        usernameLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(usernameLabel, gbc);
        gbc.gridx++;
        JTextField usernameField = new JTextField(15);
        usernameField.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(usernameField, gbc);

        // Password field
        gbc.gridx = 0;
        gbc.gridy++;
        JLabel passwordLabel = new JLabel("Password:");
        passwordLabel.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(passwordLabel, gbc);
        gbc.gridx++;
        JPasswordField passwordField = new JPasswordField(15);
        passwordField.setFont(new Font("Arial", Font.PLAIN, 18));
        loginDetailsPanel.add(passwordField, gbc);

        // Show Password checkbox
        gbc.gridx = 0;
        gbc.gridy++;
        JCheckBox showPasswordCheckbox = new JCheckBox("Show Password");
        showPasswordCheckbox.setBackground(new Color(252, 252, 252));
        showPasswordCheckbox.setFont(new Font("Arial", Font.PLAIN, 16));
        showPasswordCheckbox.addActionListener(e -> {
            if (showPasswordCheckbox.isSelected()) {
                passwordField.setEchoChar((char) 0);
            } else {
                passwordField.setEchoChar('*');
            }
        });
        loginDetailsPanel.add(showPasswordCheckbox, gbc);

        // Login button
        gbc.gridx = 0;
        gbc.gridy++;
        gbc.gridwidth = 2;
        JButton loginButton = new JButton("Login");
        loginButton.setBackground(new Color(0, 150, 136));
        loginButton.setForeground(Color.WHITE);
        loginButton.setFont(new Font("Arial", Font.BOLD, 18));
        loginButton.addActionListener(e -> {
            String username = usernameField.getText();
            String password = new String(passwordField.getPassword());
            boolean isAdmin = adminRadioButton.isSelected(); // Check if Admin radio button is selected

            if (authenticate(username, password)) {
                currentUser = username;
                CardLayout cardLayout = (CardLayout) cards.getLayout();
                cardLayout.show(cards, HOTEL_PANEL);
            } else {
                JOptionPane.showMessageDialog(null, "Invalid username or password. Please try again.");
            }
        });
        loginDetailsPanel.add(loginButton, gbc);

        // Signup button
        gbc.gridy++;
        JButton signupButton = new JButton("Sign Up");
        signupButton.setBackground(new Color(0, 172, 193));
        signupButton.setForeground(Color.WHITE);
        signupButton.setFont(new Font("Arial", Font.BOLD, 18));
        signupButton.addActionListener(e -> showSignupDialog());
        loginDetailsPanel.add(signupButton, gbc);

        // Add user type and login details panels to login panel
        loginPanel.add(userTypePanel, BorderLayout.NORTH);
        loginPanel.add(loginDetailsPanel, BorderLayout.CENTER);

        return loginPanel;
    }



    private Object showSignupDialog() {
		// TODO Auto-generated method stub
		return null;
	}

	private boolean authenticate(String username, String password) {
		// TODO Auto-generated method stub
		return false;
	}

	private void showAddPersonsDialog1(int roomNumber) {
		// TODO Auto-generated method stub
		
	}

	private void handleButtonClick(String buttonText) {
        switch (buttonText) {
            case "Available Rooms":
                displayAvailableRooms();
                break;
            case "Cancel Booking":
                cancelBooking();
                break;
            case "Exit":
                exitProgram();
                break;
            default:
                break;
        }
    }

	private void displayAvailableRooms() {
	    StringBuilder availableRooms = new StringBuilder("Available Rooms:\n");
	    for (Room room : hotel.getRooms()) {
	        if (!room.isBooked()) {
	            availableRooms.append("Room ").append(room.getRoomNumber()).append(" - Max Persons: ").append(room.getMaxPersons());
	            availableRooms.append(" - Price: ").append(room.getPrice()).append("\n"); // Add price information
	        } else {
	            availableRooms.append("Room ").append(room.getRoomNumber()).append(" - BOOKED\n");
	        }
	    }
	    outputArea.setText(availableRooms.toString());
	}


    private void cancelBooking() {
        String input = JOptionPane.showInputDialog(null, "Enter room number to cancel booking:");
        try {
            if (input != null) {
                int roomNumber = Integer.parseInt(input.trim());
                hotel.cancelBooking(roomNumber);
                displayAvailableRooms(); // Update the list of available rooms after canceling booking
            }
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(null, "Invalid room number. Please try again.");
        }
    }

    private void exitProgram() {
        int choice = JOptionPane.showConfirmDialog(null, "Are you sure you want to exit?", "Exit Confirmation", JOptionPane.YES_NO_OPTION);
        if (choice == JOptionPane.YES_OPTION) {
            System.exit(0);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(HotelBookingSystem::new);
    }
}

class User implements Serializable {
    private String username;
    private String password;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}

class Room implements Serializable {
    private int roomNumber;
    private boolean isBooked;
    private int maxPersons;
    private int price; // Price per night based on maxPersons

    public Room(int roomNumber, int maxPersons) {
        this.roomNumber = roomNumber;
        this.isBooked = false;
        this.maxPersons = maxPersons;
        this.price = calculatePrice(); // Set price based on maxPersons
    }

    public int getRoomNumber() {
        return roomNumber;
    }

    public boolean isBooked() {
        return isBooked;
    }

    public void bookRoom() {
        isBooked = true;
    }

    public void cancelBooking() {
        isBooked = false;
    }

    public int getMaxPersons() {
        return maxPersons;
    }

    public int getPrice() {
        return price;
    }

    private int calculatePrice() {
        if (maxPersons == 2) {
            return 2500; // Price for room good for 2 people
        } else if (maxPersons == 5) {
            return 4000; // Price for room good for 5 people
        }
        return 0; // Default price
    }
}

class Hotel implements Serializable {
    private ArrayList<Room> rooms;

    public Hotel(int numRooms) {
        this.rooms = new ArrayList<>();
        for (int i = 1; i <= numRooms; i++) {
            if (i <= 5) {
                rooms.add(new Room(i, 2)); // Rooms 1-5 for 2 persons
            } else {
                rooms.add(new Room(i, 5)); // Rooms 6-10 for 5 persons
            }
        }
    }

    public ArrayList<Room> getRooms() {
        return rooms;
    }

    public void bookRoom(int roomNumber) {
        for (Room room : rooms) {
            if (room.getRoomNumber() == roomNumber && !room.isBooked()) {
                room.bookRoom();
                int persons = 1; // Assume only the person making the booking by default
                if (!indicateBooking(roomNumber, persons)) {
                    JOptionPane.showMessageDialog(null, "Failed to indicate booking for Room " + roomNumber + ".");
                    return;
                } else {
                    int price = room.getPrice();
                    JOptionPane.showMessageDialog(null, "Room " + roomNumber + " booked successfully.\n" +
                            "Total Price: " + price + "\n" +
                            "Please pay over the counter.");
                }
                return;
            }
        }
        JOptionPane.showMessageDialog(null, "Room " + roomNumber + " is not available.");
    }


    private boolean indicateBooking(int roomNumber, int persons) {
        String directoryPath = "C:\\Users\\PC\\Desktop\\HOTEL SYSTEM";
        File directory = new File(directoryPath);
        if (!directory.exists()) {
            if (!directory.mkdirs()) {
                JOptionPane.showMessageDialog(null, "Failed to create directory: " + directoryPath);
                return false;
            }
        }

        String filePath = directoryPath + "\\Room" + roomNumber + "_Booked.txt";
        try (PrintWriter writer = new PrintWriter(filePath)) {
            writer.println("Room Number: " + roomNumber);
            writer.println("Number of Persons: " + persons);
            System.out.println("Booking details saved to file: " + filePath);
            return true;
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "An error occurred while writing to the file: " + e.getMessage());
            e.printStackTrace();
            return false;
        }
    }
    
    
  
    public Room getRoomByNumber(int roomNumber) {
        for (Room room : rooms) {
            if (room.getRoomNumber() == roomNumber) {
                return room;
            }
        }
        return null; // Room not found
    }


    public void cancelBooking(int roomNumber) {
        for (Room room : rooms) {
            if (room.getRoomNumber() == roomNumber && room.isBooked()) {
                room.cancelBooking();
                if (!removeBookingIndicator(roomNumber)) {
                    JOptionPane.showMessageDialog(null, "Failed to remove booking indicator for Room " + roomNumber + ".");
                } else {
                    JOptionPane.showMessageDialog(null, "Booking for Room " + roomNumber + " cancelled.");
                }
                return;
            }
        }
        JOptionPane.showMessageDialog(null, "No booking found for Room " + roomNumber + ".");
    }

    private boolean removeBookingIndicator(int roomNumber) {
        String directoryPath = "C:\\Users\\PC\\Desktop\\HOTEL SYSTEM";
        String filePath = directoryPath + "\\Room" + roomNumber + "_Booked.txt";
        File file = new File(filePath);
        if (file.exists()) {
            if (file.delete()) {
                System.out.println("Booking indicator file deleted: " + file.getName());
                return true;
            } else {
                System.out.println("Failed to delete booking indicator file.");
                return false;
            }
        } else {
            System.out.println("Booking indicator file does not exist.");
            return true; // If the file doesn't exist, consider it as a success since the booking is already cancelled.
        }
    }

    public void addPersonsToRoom(int roomNumber, int persons) {
        for (Room room : rooms) {
            if (room.getRoomNumber() == roomNumber) {
                int totalPersons = persons > 0 ? persons + 1 : 1; // Include the person making the booking if any additional persons are added

                room.bookRoom(); // Mark the room as booked

                if (!indicateBooking(roomNumber, totalPersons)) {
                    JOptionPane.showMessageDialog(null, "Failed to indicate booking for Room " + roomNumber + ".");
                } else {
                    JOptionPane.showMessageDialog(null, "Room " + roomNumber + " booked successfully. " + totalPersons + " person(s) added to Room " + roomNumber + ".");
               	
                }
                
                return;
            }
        }
        JOptionPane.showMessageDialog(null, "No booking found for Room " + roomNumber + ".");
    }



    public String getAvailableRooms() {
        StringBuilder availableRooms = new StringBuilder("Available Rooms:\n");
        for (Room room : rooms) {
            if (!room.isBooked()) {
                availableRooms.append("Room ").append(room.getRoomNumber()).append(" - Max Persons: ").append(room.getMaxPersons()).append("\n");
            } else {
                availableRooms.append("Room ").append(room.getRoomNumber()).append(" - BOOKED\n");
            }
        }
        return availableRooms.toString();
    }
}
