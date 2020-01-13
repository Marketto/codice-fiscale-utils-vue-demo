<template>
  <div class="container">
    <div class="row">
      <b-form-group
        :state="stateCf"
        description="Codice fiscale"
        label="Codice Fiscale"
        label-for="cf"
        :invalid-feedback="invalidCf"
        class="col-12"
      >
        <b-form-input :state="stateCf" id="cf" v-model="cf" name="codiceFiscale"></b-form-input>
      </b-form-group>

      <hr class="col-12">

      <b-form-group
        :state="stateLastName"
        description="Last Name"
        label="Last Name"
        label-for="lastName"
        :invalid-feedback="invalidLastName"
        class="col-md-6 col-sm-12"
      >
        <b-form-input :state="stateLastName" id="lastName" v-model="lastName" trim name="lastName"></b-form-input>
      </b-form-group>

      <b-form-group
        :state="stateFirstName"
        description="First Name"
        label="First Name"
        label-for="name"
        :invalid-feedback="invalidFirstName"
        class="col-md-6 col-sm-12"
      >
        <b-form-input :state="stateFirstName" id="firstName" v-model="firstName" trim name="firstName"></b-form-input>
      </b-form-group>

      <b-form-group
        :state="stateDate"
        description="Birth Date"
        label="Birth Date"
        label-for="date"
        :invalid-feedback="invalidDate"
        class="col-md-6 col-sm-6"
      >
        <b-form-input :state="stateDate" id="date" v-model="date" trim type="date" name="birthDate"></b-form-input>
      </b-form-group>

      <b-form-group
        :state="stateGender"
        description="Gender"
        label="Select gender"
        :invalid-feedback="invalidGender"
        class="col-md-6 col-sm-6"
      >
        <div class="container">
          <div class="row">
            <b-form-radio class="col-6" v-model="gender" name="gender" value="M">Male</b-form-radio>
            <b-form-radio class="col-6" v-model="gender" name="gender" value="F">Female</b-form-radio>
          </div>
        </div>
      </b-form-group>
      <b-form-group
        :state="stateArea"
        description="Birth Area"
        label="Birth Area"
        label-for="area"
        :invalid-feedback="invalidArea"
        class="col-md-4 col-sm-5"
      >
        <b-form-input list="area-list" v-model="area" :state="stateArea" name="birthArea"></b-form-input>
        <datalist id="area-list">
          <option v-for="entry in areas" v-bind:key="entry">{{ entry }}</option>
        </datalist>
      </b-form-group>

      <b-form-group
        :state="statePlace"
        description="Birth Place"
        label="Birth Place"
        label-for="place"
        :invalid-feedback="invalidPlace"
        class="col-md-8 col-sm-7"
      >
        <b-form-input list="place-list" v-model="place" :state="statePlace" name="birthPlace"></b-form-input>
        <datalist id="place-list">
          <option
            v-for="entry in places(cf, date, area, place)"
            v-bind:key="entry.codiceBelfiore"
          >{{ entry.name }}</option>
        </datalist>
      </b-form-group>
    </div>
  </div>
</template>

<script>
import { Belfiore, Parser, Validator } from "@marketto/codice-fiscale-utils";
const FOREIGN_COUNTRY = "Foreign Country";
export default {
  name: "CodiceFiscale",
  computed: {
    area: {
      get() {
        return this.pArea;
      },
      set(area) {
        if (this.pArea !== area) {
          this.place = null;
          this.pArea = area;
        }
      }
    },
    cf: {
      get() {
        return this.codiceFiscale;
      },
      set(cf) {
        this.codiceFiscale = cf;
        if (cf) {
          if (!this.date) {
            const date = Parser.cfToBirthDate(cf);
            if (date) {
              this.date = date.toJSON().substr(0, 10);
            }
          }
          if (!this.gender) {
            this.gender = Parser.cfToGender(cf);
          }
          if (!this.place && !this.area) {
            const place = Parser.cfToBirthPlace(cf);
            if (place) {
              this.area = place.province || FOREIGN_COUNTRY;
              this.place = place.name;
            }
          }
        }
      }
    },
    stateCf() {
      if (!this.cf) {
        return null;
      }
      return (
        Validator.codiceFiscale(this.cf).valid &&
        !Validator.codiceFiscale(this.cf).mismatchPersonalInfo({
          lastName: this.lastName,
          firstName: this.firstName,
          date: this.date,
          gender: this.gender,
          place: this.place
        })
      );
    },
    stateLastName() {
      if (!this.lastName) {
        return null;
      }
      return (
        Validator.isLastNameValid(this.lastName) &&
        !Validator.codiceFiscale(this.cf).mismatchLastName(this.lastName)
      );
    },
    stateFirstName() {
      if (!this.firstName) {
        return null;
      }
      return (
        Validator.isFirstNameValid(this.firstName) &&
        !Validator.codiceFiscale(this.cf).mismatchFirstName(this.firstName)
      );
    },
    stateDate() {
      if (!this.date) {
        return null;
      }
      return (
        Validator.isBirthDateValid(this.date) &&
        !Validator.birthDatePlaceMismatch(this.date, this.place) &&
        !Validator.codiceFiscale(this.cf).mismatchBirthDate(this.date)
      );
    },
    stateGender() {
      if (!this.gender) {
        return null;
      }
      return (
        Validator.isGenderValid(this.gender) &&
        !Validator.codiceFiscale(this.cf).mismatchGender(this.gender)
      );
    },
    stateArea() {
      return null;
    },
    statePlace() {
      if (!this.place) {
        return null;
      }
      return (
        Validator.isBirthPlaceValid(this.place) &&
        !Validator.birthPlaceDateMismatch(this.place, this.date) &&
        !Validator.codiceFiscale(this.cf || "").mismatchBirthPlace(this.place)
      );
    },
    invalidCf() {
      if ((this.cf || "").length < 16) {
        return "Please enter a full 16 chars Codice Fiscale";
      }
      if ((this.cf || "").length > 16) {
        return `You wrote ${
          this.cf.length
        } chars, a valid CF should be 16 chars only`;
      }
      if (Validator.codiceFiscale(this.cf).invalid) {
        return "Please enter a valid Codice Fiscale";
      }
      const fieldChecker = {
        lastName: this.stateLastName,
        firstName: this.stateFirstName,
        date: this.stateDate,
        gender: this.stateGender,
        place: this.statePlace
      };
      const fieldsToCheck = Object.entries(fieldChecker)
        .filter(([, check]) => check === false)
        .map(([key]) => key);
      return `The Codice Fiscale doesn't match ${fieldsToCheck.join(", ")}`;
    },
    invalidLastName: () =>
      "Provided Last name is not valid or doesn't match the current Codice Fiscale",
    invalidFirstName: () =>
      "Provided First name is not valid or doesn't match the current Codice Fiscale",
    invalidDate() {
      if (Validator.isBirthDateInvalid(this.date)) {
        return "Please enter a valid date";
      }
      return "Provided date doesn't match the current Codice Fiscale";
    },
    invalidGender: () =>
      "Provided gender doesn't match the current Codice Fiscale",
    invalidArea: () => null,
    invalidPlace() {
      if (Validator.isBirthPlaceInvalid(this.place)) {
        return "Please enter a valid city or country name";
      }
      if (Validator.birthPlaceDateMismatch(this.place, this.date)) {
        return "Provided place seems to be expired or not yet founded for the given birth date";
      }
      return "Provided place doesn't match the current Codice Fiscale";
    }
  },
  data() {
    return {
      codiceFiscale: null,
      lastName: null,
      firstName: null,
      date: null,
      gender: null,
      pArea: null,
      place: null,
      areas: [FOREIGN_COUNTRY, ...Belfiore.provinces],
      places(cf, date, area, place) {
        if ((place || "").length < 3 && !area) {
          return [];
        }
        let scopedBelfiore = Belfiore;
        const parsedDate = Parser.parseDate(date);
        if (parsedDate) {
          scopedBelfiore = scopedBelfiore.active(parsedDate);
        }
        if (area === FOREIGN_COUNTRY) {
          scopedBelfiore = scopedBelfiore.countries;
        } else if (area) {
          scopedBelfiore = scopedBelfiore.byProvince(area);
        }
        return ((place && scopedBelfiore.searchByName(place)) || []).sort(
          (a, b) => (a.name > b.name ? 1 : b.name > a.name ? -1 : 0)
        );
      }
    };
  }
};
</script>
