import java.util.*;

class Account {
    protected String accNo, name, address, phNo;
    protected float balance, existingLoan;

    public Account(String accNo, String name, String address, String phNo, float balance, float existingLoan) {
        this.accNo = accNo;
        this.name = name;
        this.address = address;
        this.phNo = phNo;
        this.balance = balance;
        this.existingLoan = existingLoan;
    }

    public String getName() { return name; }
    public float getBalance() { return balance; }
    public float getExistingLoan() { return existingLoan; }
    public void displayDetails() {
        System.out.println("\n--- Account Holder Details ---");
        System.out.println("Account No: " + accNo);
        System.out.println("Name: " + name);
        System.out.println("Address: " + address);
        System.out.println("Phone: " + phNo);
        System.out.printf("Balance: ₹%.2f\n", balance);
        System.out.printf("Existing Loan: ₹%.2f\n", existingLoan);
    }
}

class SavingAccount extends Account {
    public SavingAccount(String accNo, String name, String address, String phNo, float balance, float existingLoan) {
        super(accNo, name, address, phNo, balance, existingLoan);
    }

    public void deposit(float amount) {
        if (amount > 0) {
            balance += amount;
            System.out.printf("\u20b9%.2f deposited successfully.\n", amount);
        } else {
            System.out.println("Invalid deposit amount.");
        }
    }

    public void withdraw(float amount) {
        if (amount <= 0) {
            System.out.println("Invalid withdrawal amount.");
        } else if (balance >= amount) {
            balance -= amount;
            System.out.printf("\u20b9%.2f withdrawn successfully.\n", amount);
        } else {
            System.out.println("Insufficient balance.");
        }
    }

    public void fixedDeposit(float amount, int years) {
        final float INTEREST_RATE = 5.0f;
        if (amount <= 0 || years <= 0) {
            System.out.println("Invalid fixed deposit details.");
            return;
        }
        double maturityAmount = amount * Math.pow((1 + INTEREST_RATE / 100), years);
        balance += maturityAmount;
        System.out.println("Fixed Deposit Successful!");
        System.out.printf("Deposited ₹%.2f for %d years.\n", amount, years);
        System.out.printf("Maturity Amount: ₹%.2f\n", maturityAmount);
    }
}

class LoanAccount extends Account {
    public LoanAccount(String accNo, String name, String address, String phNo, float balance, float existingLoan) {
        super(accNo, name, address, phNo, balance, existingLoan);
    }

    public void payEMI(float emiAmount) {
        if (emiAmount <= 0) {
            System.out.println("Invalid EMI amount.");
        } else {
            balance -= emiAmount;
            System.out.printf("\u20b9%.2f EMI paid successfully.\n", emiAmount);
        }
    }

    public void topUpLoan(float amount, int tenure, float interestRate) {
        if (amount <= 0 || tenure <= 0 || interestRate <= 0) {
            System.out.println("Invalid loan details!");
            return;
        }

        if (existingLoan == 0) {
            System.out.println("No existing loan found! Not eligible for top-up.");
            return;
        }

        float interest = (amount * interestRate * tenure) / 100;
        float totalRepayment = amount + interest;
        existingLoan += amount;
        balance += amount;

        System.out.println("Top-Up Loan Approved!");
        System.out.printf("Loan Amount: ₹%.2f | Interest: %.2f%% | Tenure: %d years\n", amount, interestRate, tenure);
        System.out.printf("Total Repayment: ₹%.2f\n", totalRepayment);
        System.out.printf("Updated Loan: ₹%.2f | Updated Balance: ₹%.2f\n", existingLoan, balance);
    }
}

public class Accountmain {
    static Scanner sc = new Scanner(System.in);
    static HashMap<String, SavingAccount> accounts = new HashMap<>();
    static HashMap<String, LoanAccount> loans = new HashMap<>();

    public static void main(String[] args) {
        // Dummy accounts
        accounts.put("Milan", new SavingAccount("SAV1234", "Milan", "Amravati", "9876543210", 10000, 0));
        accounts.put("Raj", new SavingAccount("SAV5678", "Raj", "Pune", "9123456789", 8000, 0));

        System.out.print("Enter your name to locate your account: ");
        String userName = sc.next();

        SavingAccount sa = accounts.get(userName);
        if (sa == null) {
            System.out.println("Account not found!");
            return;
        }

        System.out.print("Do you want to activate loan features? (yes/no): ");
        String loanOption = sc.next().toLowerCase();
        LoanAccount la = null;
        if (loanOption.equals("yes")) {
            la = new LoanAccount("LOAN" + sa.accNo.substring(3), sa.name, sa.address, sa.phNo, sa.getBalance(), 1000000);
            loans.put(userName, la);
        }

        int choice;
        do {
            displayMainMenu();
            choice = sc.nextInt();
            switch (choice) {
                case 1 -> sa.displayDetails();
                case 2 -> sa.deposit(getInput("Enter amount to deposit: "));
                case 3 -> sa.withdraw(getInput("Enter amount to withdraw: "));
                case 4 -> System.out.printf("Current Balance: ₹%.2f\n", sa.getBalance());
                case 5 -> handleFixedDeposit(sa);
                case 6 -> {
                    if (la != null) handleLoanMenu(la);
                    else System.out.println("Loan features not activated.");
                }
                case 7 -> System.out.println("Thank you for banking with Super Bank!");
                default -> System.out.println("Invalid choice. Try again.");
            }
        } while (choice != 7);
    }

    static void displayMainMenu() {
        System.out.println("\n========== Super Bank Main Menu ==========");
        System.out.println("1. View Account Details");
        System.out.println("2. Deposit Money");
        System.out.println("3. Withdraw Money");
        System.out.println("4. Check Balance");
        System.out.println("5. Fixed Deposit");
        System.out.println("6. Loan Menu");
        System.out.println("7. Exit");
        System.out.print("Enter your choice: ");
    }

    static void handleFixedDeposit(SavingAccount sa) {
        float amount = getInput("Enter FD amount: ");
        System.out.print("Enter duration (years): ");
        int years = sc.nextInt();
        sa.fixedDeposit(amount, years);
    }

    static void handleLoanMenu(LoanAccount la) {
        int loanChoice;
        do {
            System.out.println("\n------ Loan Menu ------");
            System.out.println("1. View Loan Details");
            System.out.println("2. Pay EMI");
            System.out.println("3. Apply Top-Up Loan");
            System.out.println("4. Return to Main Menu");
            System.out.print("Enter your choice: ");
            loanChoice = sc.nextInt();

            switch (loanChoice) {
                case 1 -> System.out.printf("Current Loan: ₹%.2f | Balance: ₹%.2f\n", la.getExistingLoan(), la.getBalance());
                case 2 -> la.payEMI(getInput("Enter EMI amount: "));
                case 3 -> handleTopUpLoan(la);
                case 4 -> System.out.println("Returning to Main Menu...");
                default -> System.out.println("Invalid choice. Try again.");
            }
        } while (loanChoice != 4);
    }

    static void handleTopUpLoan(LoanAccount la) {
        float amt = getInput("Enter Loan Amount: ");
        System.out.print("Enter Tenure (years): ");
        int yrs = sc.nextInt();
        System.out.print("Enter Interest Rate (%): ");
        float rate = sc.nextFloat();
        la.topUpLoan(amt, yrs, rate);
    }

    static float getInput(String message) {
        System.out.print(message);
        return sc.nextFloat();
    }
}
