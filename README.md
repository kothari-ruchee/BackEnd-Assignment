# Back-end demo project

The goal of this demo-project is to build a GraphQL service for space flight booking.


## Service 

- The service is developed with Node.js.
- Postgres database for storage. 
- Docker so anyone can run the project directly.
- Git repository as a zip file.

## Technical specifications

### The server

- `koa` to create the server.
- `apollo-server` to add the graphql endpoint to the server.
- The server runs on port `3000`.
- The graphql endpoint are accessible at `/graphql`.

### Libraries

-  `knex` for database migration and database querying


### Data

sample files with some data to help you get started:

- `./planets.json` contains the available planets.
- `./space-centers.json` contains a list of space centers and their planet's code.
- `./bookings.json` dummy bookings information.
- `./flights.json` dummy flights information.


## Functional specifications

This service is supposed to be used to get informations about planets, space centers, flights and bookings and to schedule and book flights.

### Models

**Planet**

| Field | Type    |
| ----- | ------- |
| id    | integer |
| name  | string  |
| code  | string  |

**Space Center**

| Field       | Type    |
| ----------- | ------- |
| id          | integer |
| uid         | string  |
| name        | string  |
| description | string  |
| latitude    | float   |
| longitude   | float   |

_Relations_:

- **A Planet has many Space centers**

**Flight**

| Field        | Type     |
| ------------ | -------- |
| id           | integer  |
| code         | string   |
| departure_at | datetime |
| seat_count   | integer  |

_Relations_:

- **A Flight has a launching site (space center)**
- **A Flight has a landing site (space center)**

**Booking**

| Field      | Type    |
| ---------- | ------- |
| id         | integer |
| seat_count | integer |
| email      | string  |

_Relations_:

- **A Booking has a Flight**

### GraphQL service

#### Types

- **Planet**

  - `id`
  - `name`
  - `code` : 3 letter planet code.
  - `spaceCenters`: Returns the list of **`SpaceCenter`** on this **`Planet`**.

    **arguments**:

    - `limit`: limit the number of items returned (default: 5, max: 10).

- **SpaceCenter**:

  - `id`
  - `uid`: a unique identifier.
  - `name`
  - `description`
  - `planet`
  - `latitude`
  - `longitude`

- **Flight**

  - `id`
  - `code`: a unique 16 byte hexadecimal code generated for each flight.
  - `launchSite`: the **`SpaceCenter`** where the rocket will launch.
  - `landingSite`: the **`SpaceCenter`** where the rocket will land.
  - `departureAt`: the ISO datetime of departure (e.g: 1970-01-01T00:00:00Z).
  - `seatCount`: the number of total seats on the flight.
  - `availableSeats`: the count of availabe seats at the current time.

- **Booking fields**

  - `id`
  - `flight` : the related **`Flight`**.
  - `seatCount` : number of seats booked.
  - `email` : user email.

#### Queries

- `planets`: Returns the list of all planets.

  **Example query**

  ```graphql
  query planets {
    planets {
      id
      name
      code
      spaceCenters(limit: 3) {
        id
      }
    }
  }
  ```

- `spaceCenters`: Returns a paginated list of space centers.

  **arguments**:

  - `page`: page index (starting at 1) (default: 1, min: 1).
  - `pageSize`: number of items returned per page (default: 10, min: 1, max: 100).

  **Example query**

  ```graphql
  query spaceCenters {
    spaceCenters {
      pagination {
        total
        page
        pageSize
      }
      nodes {
        id
        uid
        name
        description
        latitude
        longitude
        planet {
          id
          name
          code
        }
      }
    }
  }
  ```

- `spaceCenter`: Returns a space center.

  **arguments**:

  - `id` : id of the space center.
  - or `uid`: uid of the space center.

  **Example Query**

  ```graphql
  query spaceCenter {
    spaceCenter(uid: "e28f6ef2-62ab-4a22-a778-bef1b32900a6") {
      id
      uid
      name
      description
      planet {
        id
        name
        code
      }
    }
  }
  ```

- `flights`: Returns a list of **`Flight`**.

  **arguments**:

  - `from`: id of a **`SpaceCenter`**.
  - `to`: id of a **`SpaceCenter`**.
  - `seatCount`: number of seats required.
  - `departureDay`: ISO date (e.g 1970-01-01).
  - `page` : deafult: 1, min: 1.
  - `pageSize`: deafult: 10, min: 1 max: 100.

  **Example query**

  ```graphql
  query flights {
    flights(pageSize: 1, page: 3) {
      pagination {
        total
        page
        pageSize
      }
      nodes {
        id
        code
        launchSite {
          name
          planet {
            name
          }
        }
        landingSite {
          name
          planet {
            name
          }
        }
      }
    }
  }
  ```

- `flight`: a **`Flight`**

  **arguments**:

  - `id`: **`Flight`** id

  **Example query**

  ```graphql
  query flight {
    flight(id: 1) {
      id
      code
      launchSite {
        name
        planet {
          name
        }
      }
      landingSite {
        name
        planet {
          name
        }
      }
    }
  }
  ```

- `bookings`: Returns a list of **`Booking`**.

  **arguments**:

  - `email`: use email.
  - `page`: page number (default: 1, min: 1).
  - `pageSize`: number of items returned (default: 10, min: 1, max: 100).

  **Example Query**

  ```graphql
  query bookings {
    bookings(email: "test@abc.io", page: 1) {
      pagination {
        total
        page
        pageSize
      }
      nodes {
        id
        seatCount
        flight {
          code
        }
      }
    }
  }
  ```

- `booking`: Returns a **`Booking`** by `id`.

  **arguments**:

  - `id`: **`Booking`** id.

  **Example query**

  ```graphql
  query booking {
    booking(id: 1) {
      id
      flight {
        code
        landingSite {
          uid
        }
      }
    }
  }
  ```

#### Mutations

- `scheduleFlight`: Create a **`Flight`**.

  **arguments**:

  - `flightInfo`:
    - `launchSiteId`: a **`SpaceCenter`** id.
    - `landingSiteId`: a **`SpaceCenter`** id.
    - `departureAt`: a Date & time of departure.
    - `seatCount`: the number of total seats on this **`Flight`**.

  **Example mutation**

  ```graphql
  mutation scheduleFlight($flight: ScheduleFlightInput!) {
    scheduleFlight(flightInfo: $flight) {
      id
      code
      launchSite {
        name
        planet {
          name
        }
      }
      landingSite {
        name
        planet {
          name
        }
      }
      availableSeats
      seatCount
      departureAt
    }
  }
  ```

- `bookFlight`

  **arguments**:

  - `bookingInfo`:
    - `seatCount`: number of seats to book (if available).
    - `flightId`: id of the **`Flight`** booked.
    - `email`: email address to record the booking.

  **Example mutation**

  ```graphql
  mutation book {
    bookFlight(
      bookingInfo: { seatCount: 10, flightId: 1, email: "test@abc.io" }
    ) {
      id
      flight {
        code
        availableSeats
        seatCount
      }
      email
    }
  }
  ```
