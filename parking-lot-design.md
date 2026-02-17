Parking Lot System
==================

Requirements
------------

- The parking lot has multiple parking spots, including compact, regular, and oversized spots.
- The parking lot supports parking for motorcycles, cars, and trucks.
- Customers can park their vehicles in spots assigned based on vehicle size.
- Customers receive a parking ticket with vehicle details and entry time at the entry point and pay a fee based on duration, vehicle size, and time of day at the exit point.

Key Rules and Assumptions
-------------------------

- Vehicle sizes: SMALL (motorcycle), MEDIUM (car), LARGE (truck).
- Spot sizes: COMPACT, REGULAR, OVERSIZED.
- Allocation rule: a vehicle fits in a spot of the same size or larger.
- Ticket duration is computed as `exitTime - entryTime`.
- Fare strategy may change based on time of day (e.g., peak hours).

Core Objects
------------

- Vehicle: contains vehicle number and size (SMALL, MEDIUM, LARGE).
- Parking Spot: spot details including size, floor, and occupancy status.
- Ticket: ticket number, vehicle details, parking spot, entry time, exit time, duration.
- Payment: payment status, method, amount, and receipt details linked to ticket.
- FareCalculator: calculates fee based on duration, vehicle size, and time of day.
- ParkingManager: handles spot allocation, assignment, lookup, and release.
- ParkingLot: central interface for entry, spot assignment, ticketing, fee calculation, and payment handling.

Key Flow
--------

Entry:

- Vehicle arrives at entry point.
- ParkingManager finds an available spot based on size.
- ParkingSpot is assigned and a Ticket is issued with entry time.

Exit:

- Ticket is presented at exit.
- Duration is computed and FareCalculator returns the fee.
- ParkingSpot is released and added back to availability.


Class Diagram (Go-Oriented)
---------------------------

Vehicle:

```go
type VehicleSize string

const (
    SizeSmall  VehicleSize = "SMALL"
    SizeMedium VehicleSize = "MEDIUM"
    SizeLarge  VehicleSize = "LARGE"
)

type Vehicle interface {
    GetVehicleNumber() string
    GetSize() VehicleSize
}

type car struct{}
type motorcycle struct{}
type truck struct{}
```

ParkingSpot:

```go
type ParkingSpot struct {
    spotNumber int
    size       VehicleSize
    floor      int
    occupied   bool
    vehicle    Vehicle
}

type ParkingSpotOps interface {
    IsAvailable() bool
    AssignToVehicle(vehicle Vehicle) error
    ReleaseFromVehicle()
    GetSpotNumber() int
    GetSize() string // SMALL, MEDIUM, LARGE
}

func (p *ParkingSpot) IsAvailable() bool {
    return !p.occupied
}

func (p *ParkingSpot) AssignToVehicle(v Vehicle) error {
    if p.occupied {
        return errors.New("spot already occupied")
    }
    p.vehicle = v
    p.occupied = true
    return nil
}

func (p *ParkingSpot) ReleaseFromVehicle() {
    p.vehicle = nil
    p.occupied = false
}
```

ParkingManager:

```go
type ParkingManager struct {
    available map[VehicleSize][]*ParkingSpot
    occupied  map[string]*ParkingSpot // map[vehicleNumber]*ParkingSpot
    mu        sync.RWMutex
}

type ParkingManagerOps interface {
    ParkVehicle(vehicle Vehicle) (*Ticket, error)
    UnparkVehicle(ticket *Ticket) error
    FindSpotForVehicle(vehicle Vehicle) (*ParkingSpot, error)
}

func (pm *ParkingManager) Park(v Vehicle) (*ParkingSpot, error) {
    pm.mu.Lock()
    defer pm.mu.Unlock()

    spots := pm.available[v.GetSize()]
    if len(spots) == 0 {
        return nil, errors.New("no spot available")
    }

    spot := spots[len(spots)-1]
    pm.available[v.GetSize()] = spots[:len(spots)-1]

    _ = spot.AssignToVehicle(v)
    pm.occupied[v.GetVehicleNumber()] = spot

    return spot, nil
}

func (pm *ParkingManager) Unpark(vehicleNumber string) error {
    pm.mu.Lock()
    defer pm.mu.Unlock()

    spot, exists := pm.occupied[vehicleNumber]
    if !exists {
        return errors.New("vehicle not found")
    }

    spot.ReleaseFromVehicle()
    pm.available[spot.size] = append(pm.available[spot.size], spot)

    delete(pm.occupied, vehicleNumber)
    return nil
}
```

Notes:

- Find spot from map: `O(1)`
- Mark free and add back to available list: `O(1)`

Ticket:

```go
type Ticket struct {
    ticketNumber string
    vehicle      Vehicle
    parkingSpot  *ParkingSpot
    entryTime    time.Time
    exitTime     time.Time
}

type TicketOps interface {
    GetParkingDuration() time.Duration
}
```

FareCalculator (Strategy Pattern):

```go
type FareStrategy interface {
    CalculateFee(ticket *Ticket) float64
}

type BaseFareStrategy struct{}
type PeakHourStrategy struct{}

func (b *BaseFareStrategy) CalculateFee(ticket *Ticket) float64 {
    return 10.0
}

func (p *PeakHourStrategy) CalculateFee(ticket *Ticket) float64 {
    return 20.0
}
```

ParkingLot:

```go
type ParkingLot struct {
    parkingManager *ParkingManager // pointer to keep data consistent
    fareStrategy   FareStrategy
    paymentService *PaymentService
}

type ParkingLotOps interface {
    Enter(vehicle Vehicle) (*Ticket, error)
    Exit(ticket *Ticket) error
}
```

Scalability  
-------------------------------------------

for scaling:

- Single lot: straightforward, in-memory state is fine.
- City-wide parking system:
  - Each parking lot is a microservice.
  - Central availability service aggregates and exposes spot availability.
  - Redis cache for spot availability reads.
  - Payment service is separate.

