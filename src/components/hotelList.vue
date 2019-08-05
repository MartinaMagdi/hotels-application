<template>
  <div class="hotels container">
    <div class="columns is-desktop columns-border">
      <div class="column">
        <b-field label="From" class="search-lables">
          <b-datepicker
            v-model="from"
            :min-date="new Date()"
            :max-date="to"
            placeholder="Click to select..."
            icon="calendar-today"
          ></b-datepicker>
        </b-field>
      </div>
      <div class="column">
        <b-field label="To" class="search-lables">
          <b-datepicker
            v-model="to"
            :min-date="from"
            placeholder="Click to select..."
            icon="calendar-today"
          ></b-datepicker>
        </b-field>
      </div>
      <div class="column">
        <b-button class="search-btn" @click="filterDate">Search</b-button>
      </div>
    </div>

    <div class="columns is-desktop columns-border">
      <div class="column is-3">
        <b-input
          v-model="searchHotelByName"
          placeholder="Hotel Name"
          type="search"
          icon="magnify"
          rounded
        ></b-input>
        <br />
        <b-field label="Price Filter" class="search-lables">
          <b-slider rounded v-model="filterByPrice" :max="1000" :step="10"></b-slider>
        </b-field>
      </div>
      <div class="column is-9">
        <div class="columns is-desktop">
          <div class="column is-6 total-nights">
            Total Nights:
            <span v-if="totalNights">{{ totalNights }}</span>
          </div>
          <div class="column is-6">
            <b-button @click="sortByName">Sort by Name</b-button>
            <b-button @click="sortByPrice">Sort by Price</b-button>
          </div>
        </div>

        <div class="columns is-multiline is-desktop">
          <div class="column is-5 hotel-card" v-for="(hotel, index) in filteredHotels" :key="index">
            <p>
              Name:
              <span>{{ hotel.name }}</span>
            </p>
            <p>
              Price:
              <span
                v-if="(totalNights)"
              >{{ Number((hotel.price * totalNights).toFixed(1)) }} AED</span>
              <span v-else>{{ hotel.price }} AED</span>
            </p>
            <p>
              City:
              <span>{{ hotel.city }}</span>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "hotelList",
  data() {
    return {
      hotels: [],
      filteredHotels: [],
      from: null,
      to: null,
      searchHotelByName: "",
      filterByPrice: 0,
      totalNights: 0
    };
  },

  mounted() {
    this.axios.get("https://api.myjson.com/bins/tl0bp").then(response => {
      this.hotels = this.filteredHotels = response.data.hotels;
    });
  },

  watch: {
    searchHotelByName() {
      if (this.searchHotelByName) {
        this.filteredHotels = this.hotels.filter(hotel => {
          return hotel.name
            .toLowerCase()
            .match(this.searchHotelByName.toLowerCase());
        });
      } else {
        this.filteredHotels = this.hotels;
      }
    },

    filterByPrice() {
      if (this.filterByPrice > 0) {
        this.filteredHotels = this.hotels.filter(hotel => {
          return hotel.price > this.filterByPrice;
        });
      } else {
        this.filteredHotels = this.hotels;
      }
    }
  },

  methods: {
    filterDate() {
      this.filteredHotels = this.hotels;
      var hotels = [];

      // calculating total nigths
      let date1 = new Date(this.from);
      let date2 = new Date(this.to);

      let dateMillisecond1 = Date.UTC(
        date1.getFullYear(),
        date1.getMonth(),
        date1.getDate()
      );
      let dateMillisecond2 = Date.UTC(
        date2.getFullYear(),
        date2.getMonth(),
        date2.getDate()
      );

      if (this.from && this.to) {
        this.totalNights = Math.floor(
          (dateMillisecond2 - dateMillisecond1) / (1000 * 60 * 60 * 24)
        );

        this.filteredHotels.filter(hotel => {
          for (let i = 0; i < hotel.availability.length; i++) {
            let newFromDateInMilliseconds = this.convertDateToMillisecond(
              hotel.availability[i].from
            );
            let newToDateInMilliseconds = this.convertDateToMillisecond(
              hotel.availability[i].to
            );

            if (
              newFromDateInMilliseconds >= dateMillisecond1 &&
              newToDateInMilliseconds <= dateMillisecond2
            )
              hotels.push(hotel);
          }
        });
        this.filteredHotels = hotels;
      }
    },

    convertDateToMillisecond(date) {
      let newDateFormat = date.split(/\-/);
      newDateFormat = [
        newDateFormat[1],
        newDateFormat[0],
        newDateFormat[2]
      ].join("-");
      newDateFormat = new Date(newDateFormat);
      let newToDateInMilliseconds = Date.UTC(
        newDateFormat.getFullYear(),
        newDateFormat.getMonth(),
        newDateFormat.getDate()
      );
      return newToDateInMilliseconds;
    },

    sortByName() {
      this.filteredHotels.sort((a, b) => {
        if (a.name > b.name) return 1;
        else if (b.name > a.name) return -1;
        else return 0;
      });
    },

    sortByPrice() {
      this.filteredHotels.sort((a, b) => {
        if (a.price > b.price) return 1;
        else if (b.price > a.price) return -1;
        else return 0;
      });
    }
  }
};
</script>

<style scoped>
.search-lables,
.hotel-card {
  text-align: left;
}

.hotel-card {
  border: 2px solid #c2bdbd;
  margin: 1rem;
}

.hotel-card p,
.total-nights {
  font-weight: bold;
}

.hotel-card span {
  font-weight: 100 !important;
}

.search-btn {
  width: -webkit-fill-available;
  width: -moz-available;
  margin: 2rem 0;
}

.columns-border {
  border-top: 2px solid #c2bdbd;
}
</style>
