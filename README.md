# Magic-Candy-Store

import java.util.Scanner;

public class CandyStoreSystem {
    private static final String[] membershipLevels = {"Basic", "Premium"};

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Create inventory and add candies
        Inventory inventory = new Inventory();
        inventory.addCandy(new ChocolateCandy("Chocolate Frog", "Flying", 5.00, 10));
        inventory.addCandy(new RegularCandy("Bertie Bott's Beans", "Random Effect", 3.00, 20));
        inventory.addCandy(new RegularCandy("Peppermint Toad", "Invisibility", 4.50, 5));
        inventory.addCandy(new ChocolateCandy("Exploding Bonbon", "Explosion", 7.50, 8));
        inventory.addCandy(new RegularCandy("Fizzing Whizzbees", "Levitation", 2.50, 20));

        // Create customer
        System.out.print("Enter your name: ");
        String name = scanner.nextLine();
        System.out.print("Are you a member? (yes/no): ");
        boolean isMember = scanner.nextLine().equalsIgnoreCase("yes");

        String membershipType = null;
        if (isMember) {
            System.out.println("Available Membership Levels: Basic, Premium");
            System.out.print("Enter your membership level: ");
            membershipType = scanner.nextLine();

            // Validate membership level input
            while (!isValidMembershipLevel(membershipType)) {
                System.out.println("Invalid membership level. Please choose between 'Basic' and 'Premium'.");
                System.out.print("Enter your membership level: ");
                membershipType = scanner.nextLine();
            }
        }

        Customer customer = new Customer(name, isMember, membershipType);

        boolean shopping = true;

        // Shopping loop
        while (shopping) {
            System.out.print("Filter by (Candy Type or Effect): ");
            String filterChoice = scanner.nextLine();

            if (filterChoice.equalsIgnoreCase("candy type")) {
                System.out.print("Enter the candy type you are looking for (Chocolate or Candy): ");
                String type = scanner.nextLine();
                var filteredCandies = inventory.filterCandiesByType(type);

                System.out.println("Available " + type + " candies:");
                for (Candy candy : filteredCandies) {
                    candy.displayInfo();
                }
            } else if (filterChoice.equalsIgnoreCase("effect")) {
                System.out.print("Enter the magical effect you are looking for (e.g., Flying, Invisibility, Levitation): ");
                String effect = scanner.nextLine();
                var filteredCandies = inventory.filterCandiesByEffect(effect);

                System.out.println("Available candies with the effect: " + effect);
                for (Candy candy : filteredCandies) {
                    candy.displayInfo();
                }
            } else {
                System.out.println("Invalid choice. Please select either 'candy type' or 'effect'.");
                continue;
            }

            // Select candy and add to cart
            System.out.print("Enter the candy name to add to your cart: ");
            String candyName = scanner.nextLine();
            Candy selectedCandy = inventory.findCandyByName(candyName);

            if (selectedCandy != null) {
                System.out.print("Enter the quantity: ");
                int quantity = scanner.nextInt();
                scanner.nextLine(); // Consume newline
                customer.addToCart(selectedCandy, quantity, scanner);
            } else {
                System.out.println("Candy not found. Please try again.");
                continue;
            }

            // Ask if the customer wants to continue shopping or checkout
            System.out.print("What would you like to do next? (Enter 'Continue Shopping' or 'Cart'): ");
            String choice = scanner.nextLine();
            if (choice.equalsIgnoreCase("Cart")) {
                shopping = false; // Exit the loop to proceed to checkout
            }
        }

        // Scenario 2: Checkout
        Order order = new Order();
        order.checkout(customer);
    }

    // Helper method to validate membership levels
    private static boolean isValidMembershipLevel(String level) {
        for (String validLevel : membershipLevels) {
            if (validLevel.equalsIgnoreCase(level)) {
                return true;
            }
        }
        return false;
    }
}


import java.util.HashMap;
import java.util.Scanner;

public class Customer {
    private String name;
    private boolean isMember;
    private String membershipType;
    private HashMap<Candy, Integer> cart = new HashMap<>();

    public Customer(String name, boolean isMember, String membershipType) {
        this.name = name;
        this.isMember = isMember;
        this.membershipType = membershipType;
    }

    public void addToCart(Candy candy, int quantity, Scanner scanner) {
        while (quantity > candy.getStock()) {
            System.out.printf("Not enough stock for %s. Available stock: %d. Please enter a new quantity: ",
                              candy.getName(), candy.getStock());
            quantity = scanner.nextInt();
        }
        cart.put(candy, cart.getOrDefault(candy, 0) + quantity);
        candy.setStock(candy.getStock() - quantity);
        System.out.println(quantity + " " + candy.getName() + "(s) added to cart.");
    }

    public HashMap<Candy, Integer> getCart() {
        return cart;
    }

    public boolean isMember() {
        return isMember;
    }

    public String getMembershipType() {
        return membershipType;
    }
}



import java.util.ArrayList;
import java.util.HashSet;

public class Inventory {
    private ArrayList<Candy> candies = new ArrayList<>();

    public void addCandy(Candy candy) {
        candies.add(candy);
    }

    public HashSet<Candy> filterCandiesByType(String type) {
        HashSet<Candy> filteredCandies = new HashSet<>();
        for (Candy candy : candies) {
            if (candy.getType().equalsIgnoreCase(type) && candy.getStock() > 0) {
                filteredCandies.add(candy);
            }
        }
        return filteredCandies;
    }

    public HashSet<Candy> filterCandiesByEffect(String effect) {
        HashSet<Candy> filteredCandies = new HashSet<>();
        for (Candy candy : candies) {
            if (candy.getMagicalEffect().equalsIgnoreCase(effect) && candy.getStock() > 0) {
                filteredCandies.add(candy);
            }
        }
        return filteredCandies;
    }

    public Candy findCandyByName(String name) {
        for (Candy candy : candies) {
            if (candy.getName().equalsIgnoreCase(name)) {
                return candy;
            }
        }
        return null;
    }
}



public class Candy {
    private String name;
    private String type;
    private String magicalEffect;
    private double price;
    private int stock;

    public Candy(String name, String type, String magicalEffect, double price, int stock) {
        this.name = name;
        this.type = type;
        this.magicalEffect = magicalEffect;
        this.price = price;
        this.stock = stock;
    }

    public String getName() {
        return name;
    }

    public String getType() {
        return type;
    }

    public String getMagicalEffect() {
        return magicalEffect;
    }

    public double getPrice() {
        return price;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }

    public void displayInfo() {
        System.out.printf("Candy: %s | Type: %s | Effect: %s | Price: $%.2f | Stock: %d%n",
                          name, type, magicalEffect, price, stock);
    }
}

class ChocolateCandy extends Candy {
    public ChocolateCandy(String name, String magicalEffect, double price, int stock) {
        super(name, "Chocolate", magicalEffect, price, stock);
    }

    @Override
    public void displayInfo() {
        System.out.printf("[Chocolate] %s | Effect: %s | Price: $%.2f | Stock: %d%n",
                          getName(), getMagicalEffect(), getPrice(), getStock());
    }
}

class RegularCandy extends Candy {
    public RegularCandy(String name, String magicalEffect, double price, int stock) {
        super(name, "Candy", magicalEffect, price, stock);
    }

    @Override
    public void displayInfo() {
        System.out.printf("[Candy] %s | Effect: %s | Price: $%.2f | Stock: %d%n",
                          getName(), getMagicalEffect(), getPrice(), getStock());
    }
}



public class Order {
    private static int orderCounter = 1;
    private int orderId;
    private double totalCost;

    public Order() {
        this.orderId = orderCounter++;
    }

    public void checkout(Customer customer) {
        double discount = 0.0;

        // Calculate total cost
        for (var entry : customer.getCart().entrySet()) {
            Candy candy = entry.getKey();
            int quantity = entry.getValue();
            totalCost += candy.getPrice() * quantity;
        }

        // Apply discount if the customer is a member
        if (customer.isMember()) {
            // Check membership level and apply discount
            if ("Basic".equalsIgnoreCase(customer.getMembershipType())) {
                discount = 0.05; // 5% discount for Basic members
            } else if ("Premium".equalsIgnoreCase(customer.getMembershipType())) {
                discount = 0.10; // 10% discount for Premium members
            }
            totalCost *= (1 - discount);
        }

        System.out.printf("Order #%d confirmed! Total Cost: $%.2f (Discount: %.0f%%)%n",
                          orderId, totalCost, discount * 100);
    }
}
