# Magic-Candy-Store

import java.util.HashMap;
import java.util.Scanner;

public class CandyStoreSystem {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        // Create inventory and add candies
        Inventory inventory = new Inventory();
        inventory.addCandy(new Candy("Chocolate Frog", "Chocolate", "Flying", 5.00, 10));
        inventory.addCandy(new Candy("Bertie Bott's Beans", "Candy", "Random Effect", 3.00, 20));
        inventory.addCandy(new Candy("Peppermint Toad", "Candy", "Invisibility", 4.50, 5));
        inventory.addCandy(new Candy("Exploding Bonbon", "Candy", "Explosion", 7.50, 8));
        inventory.addCandy(new Candy("Fizzing Whizzbees", "Candy", "Levitation", 2.50, 20));

        // Membership discounts
        HashMap<String, Double> membershipDiscounts = new HashMap<>();
        membershipDiscounts.put("Basic", 0.10); // 10% discount

        // Create customer
        System.out.print("Enter your name: ");
        String name = scanner.nextLine();
        System.out.print("Are you a member? (yes/no): ");
        boolean isMember = scanner.nextLine().equalsIgnoreCase("yes");
        Customer customer = new Customer(name, isMember);

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
        order.checkout(customer, membershipDiscounts);
    }
}
import java.util.HashMap;

public class Order {
    private static int orderCounter = 1;
    private int orderId;
    private double totalCost;

    public Order() {
        this.orderId = orderCounter++;
    }

    public void checkout(Customer customer, HashMap<String, Double> membershipDiscounts) {
        double discount = 0.0;

        // Calculate total cost
        for (var entry : customer.getCart().entrySet()) {
            Candy candy = entry.getKey();
            int quantity = entry.getValue();
            totalCost += candy.getPrice() * quantity;
        }

        // Apply discount if the customer is a member
        if (customer.isMember()) {
            discount = membershipDiscounts.getOrDefault("Basic", 0.0); // Example: 10% for Basic members
            totalCost *= (1 - discount);
        }

        System.out.printf("Order #%d confirmed! Total Cost: $%.2f (Discount: %.0f%%)%n",
                          orderId, totalCost, discount * 100);
    }
}
import java.util.HashMap;
import java.util.Scanner;

public class Customer {
    private String name;
    private boolean isMember;
    private HashMap<Candy, Integer> cart = new HashMap<>();

    public Customer(String name, boolean isMember) {
        this.name = name;
        this.isMember = isMember;
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

    public void displayInventory() {
        for (Candy candy : candies) {
            candy.displayInfo();
        }
    }

    public Candy findCandyByName(String name) {
        for (Candy candy : candies) {
            if (candy.getName().equalsIgnoreCase(name)) {
                return candy;
            }
        }
    }
}
public class Candy {
    private String name;
    private String type; // New parameter: Type (chocolate or candy)
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

    // Getters and Setters
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

    // Display Candy Information
    public void displayInfo() {
        System.out.printf("Candy: %s | Type: %s | Effect: %s | Price: $%.2f | Stock: %d%n",
                          name, type, magicalEffect, price, stock);
    }
}
