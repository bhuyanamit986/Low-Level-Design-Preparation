# 🚗 Project: Parking Lot System

> **Learn by Building** - A complete parking lot management system

## 📋 Requirements

### Functional Requirements
- Multi-floor parking lot
- Support for different vehicle types (Motorcycle, Car, Bus)
- Different spot sizes (Compact, Regular, Large)
- Entry/Exit ticket system
- Hourly-based pricing
- Real-time availability display

### Non-Functional Requirements
- Thread-safe operations
- Extensible for new vehicle types
- Clean, maintainable code

## 🏗️ Project Structure

```
parking-lot/
├── src/
│   ├── models/
│   │   ├── Vehicle.java
│   │   ├── ParkingSpot.java
│   │   ├── ParkingFloor.java
│   │   ├── ParkingTicket.java
│   │   └── ParkingLot.java
│   ├── enums/
│   │   ├── VehicleType.java
│   │   ├── SpotSize.java
│   │   └── SpotStatus.java
│   ├── strategies/
│   │   └── PricingStrategy.java
│   ├── exceptions/
│   │   └── ParkingExceptions.java
│   └── Main.java
└── README.md
```

## 📦 Implementation

### Step 1: Define Enums

```java
// src/enums/VehicleType.java
public enum VehicleType {
    MOTORCYCLE,
    CAR,
    BUS
}

// src/enums/SpotSize.java
public enum SpotSize {
    COMPACT(1),
    REGULAR(2),
    LARGE(3);
    
    private final int size;
    
    SpotSize(int size) {
        this.size = size;
    }
    
    public int getSize() { return size; }
    
    public boolean canFit(SpotSize required) {
        return this.size >= required.size;
    }
}

// src/enums/SpotStatus.java
public enum SpotStatus {
    AVAILABLE,
    OCCUPIED,
    RESERVED,
    OUT_OF_SERVICE
}
```

### Step 2: Create Vehicle Hierarchy

```java
// src/models/Vehicle.java
public abstract class Vehicle {
    private final String licensePlate;
    private final VehicleType type;
    
    protected Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public abstract SpotSize getRequiredSpotSize();
    
    public String getLicensePlate() { return licensePlate; }
    public VehicleType getType() { return type; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Vehicle)) return false;
        Vehicle vehicle = (Vehicle) o;
        return licensePlate.equals(vehicle.licensePlate);
    }
    
    @Override
    public int hashCode() {
        return licensePlate.hashCode();
    }
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
```

### Step 3: Create Parking Spot

```java
// src/models/ParkingSpot.java
public class ParkingSpot {
    private final String spotId;
    private final SpotSize size;
    private final int floorNumber;
    private SpotStatus status;
    private Vehicle currentVehicle;
    
    public ParkingSpot(String spotId, SpotSize size, int floorNumber) {
        this.spotId = spotId;
        this.size = size;
        this.floorNumber = floorNumber;
        this.status = SpotStatus.AVAILABLE;
        this.currentVehicle = null;
    }
    
    public synchronized boolean canFitVehicle(Vehicle vehicle) {
        return status == SpotStatus.AVAILABLE && 
               size.canFit(vehicle.getRequiredSpotSize());
    }
    
    public synchronized boolean park(Vehicle vehicle) {
        if (!canFitVehicle(vehicle)) {
            return false;
        }
        this.currentVehicle = vehicle;
        this.status = SpotStatus.OCCUPIED;
        return true;
    }
    
    public synchronized Vehicle unpark() {
        Vehicle vehicle = this.currentVehicle;
        this.currentVehicle = null;
        this.status = SpotStatus.AVAILABLE;
        return vehicle;
    }
    
    // Getters
    public String getSpotId() { return spotId; }
    public SpotSize getSize() { return size; }
    public int getFloorNumber() { return floorNumber; }
    public SpotStatus getStatus() { return status; }
    public Vehicle getCurrentVehicle() { return currentVehicle; }
    public boolean isAvailable() { return status == SpotStatus.AVAILABLE; }
}
```

### Step 4: Create Parking Floor

```java
// src/models/ParkingFloor.java
public class ParkingFloor {
    private final int floorNumber;
    private final List<ParkingSpot> spots;
    private final Map<SpotSize, List<ParkingSpot>> spotsBySize;
    
    public ParkingFloor(int floorNumber, int compactSpots, int regularSpots, int largeSpots) {
        this.floorNumber = floorNumber;
        this.spots = new ArrayList<>();
        this.spotsBySize = new EnumMap<>(SpotSize.class);
        
        initializeSpots(compactSpots, regularSpots, largeSpots);
    }
    
    private void initializeSpots(int compact, int regular, int large) {
        int spotNum = 1;
        
        List<ParkingSpot> compactList = new ArrayList<>();
        for (int i = 0; i < compact; i++) {
            ParkingSpot spot = new ParkingSpot(
                String.format("F%d-C%03d", floorNumber, spotNum++),
                SpotSize.COMPACT,
                floorNumber
            );
            spots.add(spot);
            compactList.add(spot);
        }
        spotsBySize.put(SpotSize.COMPACT, compactList);
        
        List<ParkingSpot> regularList = new ArrayList<>();
        for (int i = 0; i < regular; i++) {
            ParkingSpot spot = new ParkingSpot(
                String.format("F%d-R%03d", floorNumber, spotNum++),
                SpotSize.REGULAR,
                floorNumber
            );
            spots.add(spot);
            regularList.add(spot);
        }
        spotsBySize.put(SpotSize.REGULAR, regularList);
        
        List<ParkingSpot> largeList = new ArrayList<>();
        for (int i = 0; i < large; i++) {
            ParkingSpot spot = new ParkingSpot(
                String.format("F%d-L%03d", floorNumber, spotNum++),
                SpotSize.LARGE,
                floorNumber
            );
            spots.add(spot);
            largeList.add(spot);
        }
        spotsBySize.put(SpotSize.LARGE, largeList);
    }
    
    public Optional<ParkingSpot> findAvailableSpot(Vehicle vehicle) {
        SpotSize required = vehicle.getRequiredSpotSize();
        
        // Try exact size first
        Optional<ParkingSpot> spot = findInList(spotsBySize.get(required), vehicle);
        if (spot.isPresent()) return spot;
        
        // Try larger spots
        for (SpotSize size : SpotSize.values()) {
            if (size.canFit(required) && size != required) {
                spot = findInList(spotsBySize.get(size), vehicle);
                if (spot.isPresent()) return spot;
            }
        }
        
        return Optional.empty();
    }
    
    private Optional<ParkingSpot> findInList(List<ParkingSpot> spots, Vehicle vehicle) {
        if (spots == null) return Optional.empty();
        return spots.stream()
                    .filter(s -> s.canFitVehicle(vehicle))
                    .findFirst();
    }
    
    public Map<SpotSize, Long> getAvailability() {
        Map<SpotSize, Long> availability = new EnumMap<>(SpotSize.class);
        for (SpotSize size : SpotSize.values()) {
            long count = spotsBySize.getOrDefault(size, List.of()).stream()
                .filter(ParkingSpot::isAvailable)
                .count();
            availability.put(size, count);
        }
        return availability;
    }
    
    public int getFloorNumber() { return floorNumber; }
    public int getTotalSpots() { return spots.size(); }
}
```

### Step 5: Create Pricing Strategy

```java
// src/strategies/PricingStrategy.java
public interface PricingStrategy {
    double calculateFee(VehicleType vehicleType, Duration duration);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private final Map<VehicleType, Double> hourlyRates;
    
    public HourlyPricingStrategy() {
        this.hourlyRates = new EnumMap<>(VehicleType.class);
        hourlyRates.put(VehicleType.MOTORCYCLE, 1.0);
        hourlyRates.put(VehicleType.CAR, 2.0);
        hourlyRates.put(VehicleType.BUS, 5.0);
    }
    
    public HourlyPricingStrategy(Map<VehicleType, Double> rates) {
        this.hourlyRates = new EnumMap<>(rates);
    }
    
    @Override
    public double calculateFee(VehicleType vehicleType, Duration duration) {
        long hours = duration.toHours();
        if (hours == 0 || duration.toMinutes() % 60 > 0) {
            hours++; // Round up to next hour
        }
        return hourlyRates.getOrDefault(vehicleType, 2.0) * hours;
    }
}

public class FlatPricingStrategy implements PricingStrategy {
    private final double flatRate;
    
    public FlatPricingStrategy(double flatRate) {
        this.flatRate = flatRate;
    }
    
    @Override
    public double calculateFee(VehicleType vehicleType, Duration duration) {
        return flatRate;
    }
}
```

### Step 6: Create Parking Ticket

```java
// src/models/ParkingTicket.java
public class ParkingTicket {
    private final String ticketId;
    private final Vehicle vehicle;
    private final ParkingSpot spot;
    private final LocalDateTime entryTime;
    private LocalDateTime exitTime;
    private double fee;
    private boolean isPaid;
    
    public ParkingTicket(Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = generateTicketId();
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = LocalDateTime.now();
        this.isPaid = false;
    }
    
    private String generateTicketId() {
        return "T" + System.currentTimeMillis() + "-" + 
               String.format("%04d", new Random().nextInt(10000));
    }
    
    public Duration getParkingDuration() {
        LocalDateTime end = exitTime != null ? exitTime : LocalDateTime.now();
        return Duration.between(entryTime, end);
    }
    
    public void processPayment(PricingStrategy strategy) {
        this.exitTime = LocalDateTime.now();
        this.fee = strategy.calculateFee(vehicle.getType(), getParkingDuration());
        this.isPaid = true;
    }
    
    // Getters
    public String getTicketId() { return ticketId; }
    public Vehicle getVehicle() { return vehicle; }
    public ParkingSpot getSpot() { return spot; }
    public LocalDateTime getEntryTime() { return entryTime; }
    public LocalDateTime getExitTime() { return exitTime; }
    public double getFee() { return fee; }
    public boolean isPaid() { return isPaid; }
}
```

### Step 7: Create Parking Lot (Main Class)

```java
// src/models/ParkingLot.java
public class ParkingLot {
    private static ParkingLot instance;
    
    private final String name;
    private final String address;
    private final List<ParkingFloor> floors;
    private final Map<String, ParkingTicket> activeTickets;
    private final Map<String, ParkingSpot> vehicleToSpot;
    private PricingStrategy pricingStrategy;
    
    private ParkingLot(String name, String address) {
        this.name = name;
        this.address = address;
        this.floors = new ArrayList<>();
        this.activeTickets = new ConcurrentHashMap<>();
        this.vehicleToSpot = new ConcurrentHashMap<>();
        this.pricingStrategy = new HourlyPricingStrategy();
    }
    
    // Singleton pattern
    public static synchronized ParkingLot getInstance(String name, String address) {
        if (instance == null) {
            instance = new ParkingLot(name, address);
        }
        return instance;
    }
    
    public void addFloor(int compactSpots, int regularSpots, int largeSpots) {
        int floorNumber = floors.size() + 1;
        floors.add(new ParkingFloor(floorNumber, compactSpots, regularSpots, largeSpots));
    }
    
    public void setPricingStrategy(PricingStrategy strategy) {
        this.pricingStrategy = strategy;
    }
    
    public synchronized ParkingTicket parkVehicle(Vehicle vehicle) {
        // Check if vehicle is already parked
        if (vehicleToSpot.containsKey(vehicle.getLicensePlate())) {
            throw new VehicleAlreadyParkedException(
                "Vehicle " + vehicle.getLicensePlate() + " is already parked"
            );
        }
        
        // Find available spot
        ParkingSpot spot = findAvailableSpot(vehicle)
            .orElseThrow(() -> new ParkingFullException("No available spots for this vehicle"));
        
        // Park the vehicle
        spot.park(vehicle);
        
        // Create ticket
        ParkingTicket ticket = new ParkingTicket(vehicle, spot);
        activeTickets.put(ticket.getTicketId(), ticket);
        vehicleToSpot.put(vehicle.getLicensePlate(), spot);
        
        System.out.println("Vehicle " + vehicle.getLicensePlate() + 
                           " parked at spot " + spot.getSpotId());
        
        return ticket;
    }
    
    public synchronized double unparkVehicle(String ticketId) {
        ParkingTicket ticket = activeTickets.get(ticketId);
        if (ticket == null) {
            throw new InvalidTicketException("Invalid ticket: " + ticketId);
        }
        
        // Process payment
        ticket.processPayment(pricingStrategy);
        
        // Free the spot
        ParkingSpot spot = ticket.getSpot();
        spot.unpark();
        
        // Remove from tracking
        activeTickets.remove(ticketId);
        vehicleToSpot.remove(ticket.getVehicle().getLicensePlate());
        
        System.out.println("Vehicle " + ticket.getVehicle().getLicensePlate() + 
                           " unparked. Fee: $" + String.format("%.2f", ticket.getFee()));
        
        return ticket.getFee();
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
    
    public void displayAvailability() {
        System.out.println("\n=== Parking Lot Availability ===");
        System.out.println("Name: " + name);
        System.out.println("Address: " + address);
        
        for (ParkingFloor floor : floors) {
            System.out.println("\nFloor " + floor.getFloorNumber() + ":");
            Map<SpotSize, Long> availability = floor.getAvailability();
            for (SpotSize size : SpotSize.values()) {
                System.out.println("  " + size + ": " + availability.get(size) + " available");
            }
        }
        System.out.println("================================\n");
    }
    
    public int getTotalFloors() { return floors.size(); }
    public int getActiveTickets() { return activeTickets.size(); }
}
```

### Step 8: Main Application

```java
// src/Main.java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        // Initialize parking lot
        ParkingLot parkingLot = ParkingLot.getInstance("Downtown Parking", "123 Main St");
        
        // Add floors
        parkingLot.addFloor(20, 50, 10);  // Floor 1
        parkingLot.addFloor(20, 50, 10);  // Floor 2
        parkingLot.addFloor(30, 30, 5);   // Floor 3
        
        // Display initial availability
        parkingLot.displayAvailability();
        
        // Create vehicles
        Car car1 = new Car("ABC-1234");
        Car car2 = new Car("DEF-5678");
        Motorcycle bike1 = new Motorcycle("MOTO-001");
        Bus bus1 = new Bus("BUS-9999");
        
        // Park vehicles
        ParkingTicket ticket1 = parkingLot.parkVehicle(car1);
        ParkingTicket ticket2 = parkingLot.parkVehicle(car2);
        ParkingTicket ticket3 = parkingLot.parkVehicle(bike1);
        ParkingTicket ticket4 = parkingLot.parkVehicle(bus1);
        
        // Display availability after parking
        parkingLot.displayAvailability();
        
        // Simulate time passing
        System.out.println("Simulating 2 hours passing...\n");
        Thread.sleep(2000);  // In real scenario, this would be hours
        
        // Unpark vehicles
        double fee1 = parkingLot.unparkVehicle(ticket1.getTicketId());
        double fee2 = parkingLot.unparkVehicle(ticket4.getTicketId());
        
        // Final availability
        parkingLot.displayAvailability();
        
        System.out.println("Total fees collected: $" + String.format("%.2f", fee1 + fee2));
    }
}
```

## 🧪 Testing

```java
public class ParkingLotTest {
    
    @Test
    void shouldParkAndUnparkVehicle() {
        ParkingLot lot = ParkingLot.getInstance("Test Lot", "Test Address");
        lot.addFloor(10, 20, 5);
        
        Car car = new Car("TEST-001");
        ParkingTicket ticket = lot.parkVehicle(car);
        
        assertNotNull(ticket);
        assertEquals("TEST-001", ticket.getVehicle().getLicensePlate());
        
        double fee = lot.unparkVehicle(ticket.getTicketId());
        assertTrue(fee > 0);
    }
    
    @Test
    void shouldThrowWhenParkingLotFull() {
        ParkingLot lot = ParkingLot.getInstance("Small Lot", "Address");
        lot.addFloor(0, 1, 0);  // Only 1 regular spot
        
        lot.parkVehicle(new Car("CAR-001"));
        
        assertThrows(ParkingFullException.class, () -> {
            lot.parkVehicle(new Car("CAR-002"));
        });
    }
}
```

## 📝 Concepts Covered

- ✅ Object-Oriented Design
- ✅ SOLID Principles
- ✅ Strategy Pattern (Pricing)
- ✅ Singleton Pattern (ParkingLot)
- ✅ Thread Safety
- ✅ Enum Usage
- ✅ Exception Handling
