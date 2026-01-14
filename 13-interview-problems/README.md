# 📘 Section 13: Common LLD Interview Problems

> **Expect 1-2 of these in interviews** - Practice these thoroughly

---

## 📑 Problem List

| # | Problem | Difficulty | Key Concepts |
|---|---------|------------|--------------|
| 1 | [Parking Lot System](#1-parking-lot-system) | Medium | OOP, State, Strategy |
| 2 | [Elevator System](#2-elevator-system) | Medium | State Machine, Strategy |
| 3 | [Library Management](#3-library-management-system) | Easy | CRUD, Relationships |
| 4 | [Vending Machine](#4-vending-machine) | Easy | State Pattern |
| 5 | [BookMyShow](#5-bookmyshow-seat-booking) | Hard | Concurrency, Booking |
| 6 | [Splitwise](#6-splitwise-expense-sharing) | Medium | Graph, Settlement |
| 7 | [URL Shortener](#7-url-shortener) | Easy | Encoding, Cache |
| 8 | [Notification System](#8-notification-system) | Medium | Observer, Strategy |

---

## 1. Parking Lot System

### Requirements
- Multiple floors, each with multiple spots
- Different spot sizes (Compact, Regular, Large)
- Different vehicle types (Motorcycle, Car, Bus)
- Entry/exit with ticket system
- Hourly pricing

### Class Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           PARKING LOT CLASS DIAGRAM                                 │
├─────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                     │
│   ┌────────────────┐         ┌────────────────────┐                                 │
│   │  ParkingLot    │         │    «enum»          │                                 │
│   ├────────────────┤         │    SpotSize        │                                 │
│   │ - floors[]     │         ├────────────────────┤                                 │
│   │ - entryPanels[]│         │ COMPACT            │                                 │
│   │ - exitPanels[] │         │ REGULAR            │                                 │
│   ├────────────────┤         │ LARGE              │                                 │
│   │ + findSpot()   │         └────────────────────┘                                 │
│   │ + parkVehicle()│                                                                │
│   └───────┬────────┘         ┌────────────────────┐                                 │
│           │                  │    «abstract»      │                                 │
│           │                  │    Vehicle         │                                 │
│           ▼                  ├────────────────────┤                                 │
│   ┌────────────────┐         │ - licensePlate     │                                 │
│   │    Floor       │         │ - type             │                                 │
│   ├────────────────┤         │ + getSize()        │                                 │
│   │ - floorNumber  │         └─────────△──────────┘                                 │
│   │ - spots[]      │                   │                                            │
│   ├────────────────┤         ┌─────────┼─────────┐                                  │
│   │ + findSpot()   │         │         │         │                                  │
│   │ + getAvailable()│     ┌──┴──┐   ┌──┴──┐   ┌──┴──┐                               │
│   └───────┬────────┘     │Motor│   │ Car │   │ Bus │                                │
│           │              │cycle│   │     │   │     │                                │
│           ▼              └─────┘   └─────┘   └─────┘                                │
│   ┌────────────────┐                                                                │
│   │  ParkingSpot   │◄─────────────────────────────────────┐                         │
│   ├────────────────┤                                      │                         │
│   │ - spotId       │         ┌────────────────────┐       │                         │
│   │ - size         │         │    ParkingTicket   │       │                         │
│   │ - vehicle      │         ├────────────────────┤       │                         │
│   │ - isAvailable  │         │ - ticketId         │───────┘                         │
│   ├────────────────┤         │ - entryTime        │                                 │
│   │ + park()       │         │ - spot             │                                 │
│   │ + unpark()     │         │ - vehicle          │                                 │
│   │ + canFit()     │         │ + calculateFee()   │                                 │
│   └────────────────┘         └────────────────────┘                                 │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Implementation

```java
// Enums
public enum VehicleType {
    MOTORCYCLE, CAR, BUS
}

public enum SpotSize {
    COMPACT, REGULAR, LARGE
}

// Vehicle hierarchy
public abstract class Vehicle {
    protected String licensePlate;
    protected VehicleType type;
    
    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public abstract SpotSize getRequiredSpotSize();
    
    public String getLicensePlate() { return licensePlate; }
    public VehicleType getType() { return type; }
}

public class Motorcycle extends Vehicle {
    public Motorcycle(String licensePlate) {
        super(licensePlate, VehicleType.MOTORCYCLE);
    }
    
    @Override
    public SpotSize getRequiredSpotSize() {
        return SpotSize.COMPACT;
    }
}

public class Car extends Vehicle {
    public Car(String licensePlate) {
        super(licensePlate, VehicleType.CAR);
    }
    
    @Override
    public SpotSize getRequiredSpotSize() {
        return SpotSize.REGULAR;
    }
}

public class Bus extends Vehicle {
    public Bus(String licensePlate) {
        super(licensePlate, VehicleType.BUS);
    }
    
    @Override
    public SpotSize getRequiredSpotSize() {
        return SpotSize.LARGE;
    }
}

// Parking Spot
public class ParkingSpot {
    private final String spotId;
    private final SpotSize size;
    private final int floorNumber;
    private Vehicle parkedVehicle;
    
    public ParkingSpot(String spotId, SpotSize size, int floorNumber) {
        this.spotId = spotId;
        this.size = size;
        this.floorNumber = floorNumber;
    }
    
    public boolean isAvailable() {
        return parkedVehicle == null;
    }
    
    public boolean canFit(Vehicle vehicle) {
        return isAvailable() && size.ordinal() >= vehicle.getRequiredSpotSize().ordinal();
    }
    
    public synchronized void park(Vehicle vehicle) {
        if (!canFit(vehicle)) {
            throw new IllegalStateException("Cannot park vehicle in this spot");
        }
        this.parkedVehicle = vehicle;
    }
    
    public synchronized Vehicle unpark() {
        Vehicle vehicle = this.parkedVehicle;
        this.parkedVehicle = null;
        return vehicle;
    }
    
    // Getters
    public String getSpotId() { return spotId; }
    public SpotSize getSize() { return size; }
    public int getFloorNumber() { return floorNumber; }
}

// Parking Floor
public class ParkingFloor {
    private final int floorNumber;
    private final List<ParkingSpot> spots;
    
    public ParkingFloor(int floorNumber, int compactSpots, int regularSpots, int largeSpots) {
        this.floorNumber = floorNumber;
        this.spots = new ArrayList<>();
        
        for (int i = 0; i < compactSpots; i++) {
            spots.add(new ParkingSpot(floorNumber + "-C" + i, SpotSize.COMPACT, floorNumber));
        }
        for (int i = 0; i < regularSpots; i++) {
            spots.add(new ParkingSpot(floorNumber + "-R" + i, SpotSize.REGULAR, floorNumber));
        }
        for (int i = 0; i < largeSpots; i++) {
            spots.add(new ParkingSpot(floorNumber + "-L" + i, SpotSize.LARGE, floorNumber));
        }
    }
    
    public Optional<ParkingSpot> findAvailableSpot(Vehicle vehicle) {
        return spots.stream()
            .filter(spot -> spot.canFit(vehicle))
            .findFirst();
    }
    
    public long getAvailableSpotCount(SpotSize size) {
        return spots.stream()
            .filter(ParkingSpot::isAvailable)
            .filter(spot -> spot.getSize() == size)
            .count();
    }
}

// Parking Ticket
public class ParkingTicket {
    private final String ticketId;
    private final Vehicle vehicle;
    private final ParkingSpot spot;
    private final LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private boolean isPaid;
    
    public ParkingTicket(Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = UUID.randomUUID().toString();
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = LocalDateTime.now();
        this.isPaid = false;
    }
    
    public double calculateFee(PricingStrategy pricingStrategy) {
        if (exitTime == null) {
            exitTime = LocalDateTime.now();
        }
        long hours = ChronoUnit.HOURS.between(entryTime, exitTime);
        if (hours == 0) hours = 1; // Minimum 1 hour
        return pricingStrategy.calculatePrice(vehicle.getType(), hours);
    }
    
    public void markPaid() {
        this.isPaid = true;
    }
    
    // Getters
}

// Pricing Strategy
public interface PricingStrategy {
    double calculatePrice(VehicleType vehicleType, long hours);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private final Map<VehicleType, Double> hourlyRates;
    
    public HourlyPricingStrategy() {
        hourlyRates = Map.of(
            VehicleType.MOTORCYCLE, 1.0,
            VehicleType.CAR, 2.0,
            VehicleType.BUS, 4.0
        );
    }
    
    @Override
    public double calculatePrice(VehicleType vehicleType, long hours) {
        return hourlyRates.getOrDefault(vehicleType, 2.0) * hours;
    }
}

// Parking Lot
public class ParkingLot {
    private final String name;
    private final List<ParkingFloor> floors;
    private final Map<String, ParkingTicket> activeTickets;
    private final PricingStrategy pricingStrategy;
    
    public ParkingLot(String name, int numFloors) {
        this.name = name;
        this.floors = new ArrayList<>();
        this.activeTickets = new ConcurrentHashMap<>();
        this.pricingStrategy = new HourlyPricingStrategy();
        
        for (int i = 0; i < numFloors; i++) {
            floors.add(new ParkingFloor(i, 50, 100, 20));
        }
    }
    
    public synchronized ParkingTicket parkVehicle(Vehicle vehicle) {
        ParkingSpot spot = findAvailableSpot(vehicle)
            .orElseThrow(() -> new ParkingFullException("No available spots"));
        
        spot.park(vehicle);
        ParkingTicket ticket = new ParkingTicket(vehicle, spot);
        activeTickets.put(ticket.getTicketId(), ticket);
        
        return ticket;
    }
    
    public double unparkVehicle(String ticketId) {
        ParkingTicket ticket = activeTickets.get(ticketId);
        if (ticket == null) {
            throw new InvalidTicketException("Ticket not found");
        }
        
        double fee = ticket.calculateFee(pricingStrategy);
        ticket.getSpot().unpark();
        ticket.markPaid();
        activeTickets.remove(ticketId);
        
        return fee;
    }
    
    private Optional<ParkingSpot> findAvailableSpot(Vehicle vehicle) {
        for (ParkingFloor floor : floors) {
            Optional<ParkingSpot> spot = floor.findAvailableSpot(vehicle);
            if (spot.isPresent()) {
                return spot;
            }
        }
        return Optional.empty();
    }
    
    public Map<SpotSize, Long> getAvailability() {
        Map<SpotSize, Long> availability = new EnumMap<>(SpotSize.class);
        for (SpotSize size : SpotSize.values()) {
            long count = floors.stream()
                .mapToLong(floor -> floor.getAvailableSpotCount(size))
                .sum();
            availability.put(size, count);
        }
        return availability;
    }
}
```

---

## 2. Elevator System

### Requirements
- Multiple elevators in a building
- Optimize for wait time
- Handle requests from floors
- Direction management

### Key Classes

```java
public enum Direction {
    UP, DOWN, IDLE
}

public enum ElevatorState {
    MOVING, STOPPED, MAINTENANCE
}

public class Elevator {
    private final int id;
    private int currentFloor;
    private Direction direction;
    private ElevatorState state;
    private final Set<Integer> destinationFloors;
    private final int capacity;
    
    public Elevator(int id, int capacity) {
        this.id = id;
        this.capacity = capacity;
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
        this.state = ElevatorState.STOPPED;
        this.destinationFloors = new TreeSet<>();
    }
    
    public void addDestination(int floor) {
        destinationFloors.add(floor);
        updateDirection();
    }
    
    public void move() {
        if (destinationFloors.isEmpty()) {
            direction = Direction.IDLE;
            state = ElevatorState.STOPPED;
            return;
        }
        
        state = ElevatorState.MOVING;
        
        if (direction == Direction.UP) {
            currentFloor++;
        } else if (direction == Direction.DOWN) {
            currentFloor--;
        }
        
        if (destinationFloors.contains(currentFloor)) {
            stop();
        }
    }
    
    private void stop() {
        state = ElevatorState.STOPPED;
        destinationFloors.remove(currentFloor);
        updateDirection();
    }
    
    private void updateDirection() {
        if (destinationFloors.isEmpty()) {
            direction = Direction.IDLE;
        } else {
            int nextFloor = destinationFloors.iterator().next();
            direction = nextFloor > currentFloor ? Direction.UP : Direction.DOWN;
        }
    }
    
    public int getCurrentFloor() { return currentFloor; }
    public Direction getDirection() { return direction; }
    public ElevatorState getState() { return state; }
}

// Elevator Controller with scheduling strategy
public interface ElevatorScheduler {
    Elevator selectElevator(List<Elevator> elevators, int requestFloor, Direction direction);
}

public class NearestElevatorScheduler implements ElevatorScheduler {
    @Override
    public Elevator selectElevator(List<Elevator> elevators, int requestFloor, Direction direction) {
        return elevators.stream()
            .filter(e -> e.getState() != ElevatorState.MAINTENANCE)
            .min(Comparator.comparingInt(e -> Math.abs(e.getCurrentFloor() - requestFloor)))
            .orElseThrow(() -> new NoAvailableElevatorException());
    }
}

public class ElevatorController {
    private final List<Elevator> elevators;
    private final ElevatorScheduler scheduler;
    
    public ElevatorController(int numElevators, int capacity) {
        this.elevators = new ArrayList<>();
        for (int i = 0; i < numElevators; i++) {
            elevators.add(new Elevator(i, capacity));
        }
        this.scheduler = new NearestElevatorScheduler();
    }
    
    public void requestElevator(int floor, Direction direction) {
        Elevator elevator = scheduler.selectElevator(elevators, floor, direction);
        elevator.addDestination(floor);
    }
    
    public void selectFloor(int elevatorId, int floor) {
        elevators.get(elevatorId).addDestination(floor);
    }
}
```

---

## 3. Library Management System

### Key Classes

```java
public class Book {
    private final String isbn;
    private final String title;
    private final String author;
    private final List<BookCopy> copies;
}

public class BookCopy {
    private final String copyId;
    private final Book book;
    private BookStatus status;
    private Member borrowedBy;
}

public class Member {
    private final String memberId;
    private final String name;
    private final List<Loan> activeLoans;
    private final int maxBooksAllowed;
    
    public boolean canBorrow() {
        return activeLoans.size() < maxBooksAllowed;
    }
}

public class Loan {
    private final String loanId;
    private final BookCopy bookCopy;
    private final Member member;
    private final LocalDate borrowDate;
    private final LocalDate dueDate;
    private LocalDate returnDate;
    
    public boolean isOverdue() {
        return returnDate == null && LocalDate.now().isAfter(dueDate);
    }
    
    public double calculateFine(double finePerDay) {
        if (!isOverdue()) return 0;
        long daysOverdue = ChronoUnit.DAYS.between(dueDate, LocalDate.now());
        return daysOverdue * finePerDay;
    }
}

public class Library {
    private final Map<String, Book> catalog;
    private final Map<String, Member> members;
    private final List<Loan> loans;
    
    public Loan borrowBook(String memberId, String isbn) {
        Member member = members.get(memberId);
        if (!member.canBorrow()) {
            throw new BorrowLimitExceededException();
        }
        
        Book book = catalog.get(isbn);
        BookCopy availableCopy = book.getAvailableCopy()
            .orElseThrow(() -> new NoAvailableCopyException());
        
        Loan loan = new Loan(availableCopy, member);
        availableCopy.markBorrowed(member);
        loans.add(loan);
        member.addLoan(loan);
        
        return loan;
    }
    
    public double returnBook(String loanId) {
        Loan loan = findLoan(loanId);
        double fine = loan.calculateFine(1.0);
        loan.markReturned();
        loan.getBookCopy().markAvailable();
        return fine;
    }
}
```

---

## 4. Vending Machine

See [Section 4: State Pattern](../04-design-patterns/behavioral/README.md#4-state-pattern) for complete implementation.

---

## 5. BookMyShow Seat Booking

### Key Challenges
- Concurrent seat selection
- Temporary seat holds
- Payment timeout handling

```java
public class Show {
    private final String showId;
    private final Movie movie;
    private final Hall hall;
    private final LocalDateTime showTime;
    private final Map<String, Seat> seats;
    private final Map<String, SeatHold> activeHolds;
}

public class Seat {
    private final String seatId;
    private final SeatType type;
    private SeatStatus status;
    private final double price;
}

public class SeatHold {
    private final String holdId;
    private final List<Seat> seats;
    private final LocalDateTime expiresAt;
    private final String userId;
}

public class BookingService {
    private static final Duration HOLD_DURATION = Duration.ofMinutes(10);
    
    @Transactional
    public SeatHold holdSeats(String showId, List<String> seatIds, String userId) {
        Show show = showRepository.findById(showId);
        
        synchronized (show) {
            List<Seat> seats = seatIds.stream()
                .map(show::getSeat)
                .collect(toList());
            
            // Check all seats are available
            for (Seat seat : seats) {
                if (seat.getStatus() != SeatStatus.AVAILABLE) {
                    throw new SeatNotAvailableException(seat.getSeatId());
                }
            }
            
            // Create hold
            SeatHold hold = new SeatHold(seats, userId, LocalDateTime.now().plus(HOLD_DURATION));
            
            // Mark seats as held
            for (Seat seat : seats) {
                seat.setStatus(SeatStatus.HELD);
            }
            
            show.addHold(hold);
            return hold;
        }
    }
    
    @Transactional
    public Booking confirmBooking(String holdId, PaymentDetails payment) {
        SeatHold hold = holdRepository.findById(holdId);
        
        if (hold.isExpired()) {
            releaseHold(hold);
            throw new HoldExpiredException();
        }
        
        // Process payment
        PaymentResult result = paymentService.process(payment);
        if (!result.isSuccessful()) {
            throw new PaymentFailedException();
        }
        
        // Create booking
        Booking booking = new Booking(hold.getSeats(), result.getTransactionId());
        for (Seat seat : hold.getSeats()) {
            seat.setStatus(SeatStatus.BOOKED);
        }
        
        holdRepository.delete(hold);
        return bookingRepository.save(booking);
    }
    
    // Scheduled job to release expired holds
    @Scheduled(fixedRate = 60000)
    public void releaseExpiredHolds() {
        List<SeatHold> expiredHolds = holdRepository.findExpired();
        for (SeatHold hold : expiredHolds) {
            releaseHold(hold);
        }
    }
}
```

---

## 6. Splitwise Expense Sharing

### Key Classes

```java
public class User {
    private final String userId;
    private final String name;
    private final String email;
}

public class Expense {
    private final String expenseId;
    private final String description;
    private final Money totalAmount;
    private final User paidBy;
    private final List<Split> splits;
    private final SplitType splitType;
}

public interface Split {
    User getUser();
    Money getAmount();
}

public class EqualSplit implements Split {
    private final User user;
    private Money amount;
    
    public void setAmount(Money total, int numParticipants) {
        this.amount = total.divide(numParticipants);
    }
}

public class ExactSplit implements Split {
    private final User user;
    private final Money amount;
}

public class PercentSplit implements Split {
    private final User user;
    private final double percent;
    private Money amount;
    
    public void calculateAmount(Money total) {
        this.amount = total.multiply(percent / 100);
    }
}

public class ExpenseService {
    private final Map<String, Map<String, Money>> balances;  // userId -> (userId -> amount)
    
    public void addExpense(Expense expense) {
        User paidBy = expense.getPaidBy();
        
        for (Split split : expense.getSplits()) {
            User owes = split.getUser();
            if (!owes.equals(paidBy)) {
                // owes user owes paidBy user
                updateBalance(owes.getUserId(), paidBy.getUserId(), split.getAmount());
            }
        }
    }
    
    private void updateBalance(String fromUser, String toUser, Money amount) {
        // Simplify: if A owes B $10 and B owes A $3, result is A owes B $7
        balances.computeIfAbsent(fromUser, k -> new HashMap<>())
                .merge(toUser, amount, Money::add);
        
        // Check reverse balance and simplify
        Money reverseBalance = balances.getOrDefault(toUser, Map.of())
                                       .getOrDefault(fromUser, Money.ZERO);
        if (reverseBalance.isGreaterThan(Money.ZERO)) {
            simplifyBalances(fromUser, toUser);
        }
    }
    
    public List<Balance> getBalances(String userId) {
        List<Balance> result = new ArrayList<>();
        
        // What user owes to others
        Map<String, Money> owes = balances.getOrDefault(userId, Map.of());
        for (Map.Entry<String, Money> entry : owes.entrySet()) {
            result.add(new Balance(userId, entry.getKey(), entry.getValue(), BalanceType.OWES));
        }
        
        // What others owe to user
        for (Map.Entry<String, Map<String, Money>> entry : balances.entrySet()) {
            Money owed = entry.getValue().getOrDefault(userId, Money.ZERO);
            if (owed.isGreaterThan(Money.ZERO)) {
                result.add(new Balance(entry.getKey(), userId, owed, BalanceType.OWED));
            }
        }
        
        return result;
    }
}
```

---

## 7. URL Shortener

```java
public class URLShortener {
    private static final String BASE_URL = "http://short.url/";
    private static final String ALPHABET = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
    private static final int BASE = ALPHABET.length();
    
    private final Map<String, String> shortToLong;
    private final Map<String, String> longToShort;
    private final AtomicLong counter;
    
    public URLShortener() {
        this.shortToLong = new ConcurrentHashMap<>();
        this.longToShort = new ConcurrentHashMap<>();
        this.counter = new AtomicLong(1);
    }
    
    public String shorten(String longUrl) {
        // Check if already shortened
        if (longToShort.containsKey(longUrl)) {
            return BASE_URL + longToShort.get(longUrl);
        }
        
        // Generate new short code
        long id = counter.getAndIncrement();
        String shortCode = encode(id);
        
        shortToLong.put(shortCode, longUrl);
        longToShort.put(longUrl, shortCode);
        
        return BASE_URL + shortCode;
    }
    
    public String expand(String shortUrl) {
        String shortCode = shortUrl.replace(BASE_URL, "");
        return shortToLong.get(shortCode);
    }
    
    private String encode(long num) {
        StringBuilder sb = new StringBuilder();
        while (num > 0) {
            sb.append(ALPHABET.charAt((int) (num % BASE)));
            num /= BASE;
        }
        return sb.reverse().toString();
    }
}
```

---

## 8. Notification System

See [Projects Section](../projects/notification-system/README.md) for complete implementation.

---

## Practice Tips

1. **Clarify requirements** before coding
2. **Start with entities** and relationships
3. **Apply SOLID principles**
4. **Use design patterns** where appropriate
5. **Handle edge cases**
6. **Discuss trade-offs**

---

**Next Section: [How to Approach LLD Interviews](../14-interview-approach/README.md)** →
