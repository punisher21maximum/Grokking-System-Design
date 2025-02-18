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
