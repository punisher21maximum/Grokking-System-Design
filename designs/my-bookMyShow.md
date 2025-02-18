# Design BookMyShow

## FR
- search tickets for events
- book tickets

## NFR
- Availibility : High, especially searching and booking
- Consistency : Absolute
- Latency : Low, other than booking tickets queue

## Estimation
### Assumption
- Total Users = 300M
- Daily ticker orders = 1M
- Surge orders = 10M

### Storage
1. Non image
- Events + User's data + Booking data
- 1k events * size of each event ~1kb + 300M * 1kb + 1kb * 1M
- 1 mb + 300GB + 1TB for 3-4 years
- ~ 1TB
2. Images and Videos
- S3 and CDN

### Cache
- 20% of data = 0.2TB

 ## Approach
 1. Search
- Postgres to store event data, structured
- ElasticSearch for faster search

 2. Booking service
- Every user gets 5min window to book tickets and pay.
- If failed than tickets not booked.

 3. How to unlock seat if booking failed?
- option 1
  
set ticket status as booked, 
run cron every minute, which un-books ticket if transaction failed
why bad? cron takes 10mins to scan table, can face issues

- option2
  
mark seats as booked using Redis TTL
get available seats from db, exclude seats from Redis

 4. Handle Surge]
Virtual waiting Queue

