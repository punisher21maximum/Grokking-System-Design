# Design Uber

## FR
- User provides src-dest and gets estimated fare 
- User proceeds to book the cab (for same given src-dest)
- Drivers can accept or deny request

## NFR 
- Availability : Highly available, especially booking a cab, tracking cab. Cab history can be traded off.
- Low latency : Except on booking request, 5min to match or fail
- Consistency : 1 customer matched to 1 driver. One user not >1 cabbie. One cabbie not >1 user.

## Estimation
### Assumption
- Daily active no. of cabs = 50k
- Daily no. of bookings = 10 rides each * 50k = 5L

### Book cab request API
- 6 booking requests per second ~= 6 RPS on server

### Update location API
- 50k send update every 5 second ~= 10k RPS

### Storage
User data
- 100M user and increase by 1M every month
- 1kb for each month
- 100GB
Rides data
- 5L bookings daily
- Each booking like 1kb
- 0.5GB

### Cache
- 20% of db, to server 80% of the request
- Cache daily booking data = 0.5GB + 20% of user data = 20GB

## API
- GET /ride/get_estimate/
body = {
    "request_data": {
        "src":
        "dest":
         ...more data...
    }
}
response = {
    "estimate": {
        "Uber XL": {
            ...data...
        }
    }
}
- POST /ride/request_booking/
- PUT /ride/respond_booking_request/ # used by driver

## Storage / Tables

1. Ride
- id
- driver_id 
- src
- dest
- eta
- status

2. Driver
- id
- status

## Drawing
<img width="998" alt="image" src="https://github.com/user-attachments/assets/52f54b56-5fed-44b6-9b84-df29a3596d6f" />

## Deep Dive
### Storing location of Drivers
1. Option 1, Postgres, Quadtree
- postgres DB, 
postgis extension
geography datatype = quadtree

leaf nodes have actual lat long

other nodes are bounding boxes

each internal nodes has min max

min = min_lat_value, min_long_value
max = max_lat_value, max_long_value

drawback
- to avoid getting a skewed tree, we rebalance 
and that can take sometime O(logN) to O(N)

<img width="888" alt="image" src="https://github.com/user-attachments/assets/3f279e9b-79dc-43c4-93c7-78802294917e" />


2. Option 2, Geohashing
Option 2 : Geohashing
Plane is divided recursively into N equal parts

Leaf node has actual location

Implement using Redis, with sortedsets, and use binarysearch

<img width="417" alt="image" src="https://github.com/user-attachments/assets/8f74a1dc-d6dd-4617-bec1-b7cb097ec620" />
