<template>
  <div class="container mt-4">

    <!-- Added Loader Component -->
    <SpinnerLoader :visible="loader" />

    <!-- Assignment Form -->

    <b-card title="Assignment Form" class="mb-4 p-4 boxShadow">
      <ValidationObserver ref="observer">
        <b-form @submit.prevent="submitForm">

          <!-- Add Group -->
          <div class="text-right mb-3">
            <b-button size="sm" variant="outline-primary" @click="addGroup">+ Add Group</b-button>
          </div>

          <!-- Dynamic Groups -->
          <div v-for="(grp, idx) in groups" :key="idx" class="border rounded p-3 mb-3">
            <h5 class="mb-3">Group {{ idx + 1 }}</h5>
            <b-row>
              <!-- Name -->
              <b-col md="4" class="mb-2">
                <ValidationProvider :vid="'name-'+idx" name="Name" rules="required|alpha_spaces" v-slot="{ errors }">
                  <b-form-group label="Name">
                    <b-form-input
                      type="text"
                      v-model="grp.name"
                      placeholder="Enter full name"
                      :state="errors.length ? false : null"
                    ></b-form-input>
                    <small class="text-danger">{{ errors[0] }}</small>
                  </b-form-group>
                </ValidationProvider>
              </b-col>

              <!-- Email -->
              <b-col md="4" class="mb-2">
                <ValidationProvider :vid="'email-'+idx" name="Email" rules="required|email" v-slot="{ errors }">
                  <b-form-group label="Email">
                    <b-form-input
                      v-model="grp.email"
                      type="email"
                      placeholder="Enter email"
                      :state="errors.length ? false : null"
                    ></b-form-input>
                    <small class="text-danger">{{ errors[0] }}</small>
                  </b-form-group>
                </ValidationProvider>
              </b-col>

              <!-- Mobile -->
              <b-col md="3" class="mb-2">
                <ValidationProvider :vid="'mobile-'+idx" name="Mobile" rules="required|mobile" v-slot="{ errors }">
                  <b-form-group label="Mobile">
                    <b-form-input
                      v-model="grp.mobile"
                      maxlength="14"
                      @input="formatMobile(idx)"
                      placeholder="(123) 456-7890"
                      :state="errors.length ? false : null"
                    ></b-form-input>
                    <small class="text-danger">{{ errors[0] }}</small>
                  </b-form-group>
                </ValidationProvider>
              </b-col>

              <!-- Remove Button -->
              <b-col md="1" class="d-flex align-items-center justify-content-end">
                <b-button v-if="groups.length > 2" variant="danger" size="sm" @click="removeGroup(idx)">✕</b-button>
              </b-col>
            </b-row>
          </div>

          <!-- Other Fields -->
          <b-row>
            <!-- Birthdate -->
            <b-col md="4" class="mb-2">
              <ValidationProvider name="Birthdate" rules="required" v-slot="{ errors }">
                <b-form-group label="Birth Date">
                  <b-form-input type="date" v-model="form.birthdate" :state="errors.length ? false : null"></b-form-input>
                  <small class="text-danger">{{ errors[0] }}</small>
                </b-form-group>
              </ValidationProvider>
            </b-col>

            <!-- Gender -->
            <b-col md="4" class="mb-2">
              <ValidationProvider name="Gender" rules="required" v-slot="{ errors }">
                <b-form-group label="Gender">
                  <b-form-radio-group v-model="form.gender" :options="genderOptions" :state="errors.length ? false : null"></b-form-radio-group>
                  <small class="text-danger">{{ errors[0] }}</small>
                </b-form-group>
              </ValidationProvider>
            </b-col>

            <!-- Languages -->
            <b-col md="4" class="mb-2">
              <ValidationProvider name="Languages" rules="required" v-slot="{ errors }">
                <b-form-group label="Languages">
                  <b-form-checkbox-group v-model="form.languages" :options="languageOptions" :state="errors.length ? false : null"></b-form-checkbox-group>
                  <small class="text-danger">{{ errors[0] }}</small>
                </b-form-group>
              </ValidationProvider>
            </b-col>

            <!-- Cities -->
            <b-col md="4" class="mb-2">
              <ValidationProvider name="Cities" rules="required" v-slot="{ errors }">
                <b-form-group label="Select Cities">
                  <treeselect
                    v-model="form.cities"
                    :options="cityOptions"
                    multiple
                    placeholder="Choose cities"
                    :state="errors.length ? false : null"
                  ></treeselect>
                  <small class="text-danger">{{ errors[0] }}</small>
                </b-form-group>
              </ValidationProvider>
            </b-col>

            <!-- File Upload -->
            <b-col md="4" class="mb-2">
              <b-form-group label="Upload File (JPEG/PDF)">
                <b-form-file
                  v-model="form.file"
                  accept=".jpg,.jpeg,.pdf"
                  @change="validateFile"
                  :state="fileError ? false : null"
                ></b-form-file>
                <small class="text-danger">{{ fileError }}</small>
              </b-form-group>
            </b-col>
            

            <!-- Submit Button -->
            <b-col cols="12" class="mt-3 text-right">
              <b-button type="submit" variant="success" size="md">Submit Form</b-button>
            </b-col>
          </b-row>

        </b-form>
      </ValidationObserver>
    </b-card>

    <!-- Submitted Data Table -->
    <b-card v-if="submittedData.length" title="Submitted Data" class="p-3">
      <b-table :items="submittedData" :fields="tableFields" bordered striped small responsive></b-table>
    </b-card>
  </div>
  <!-- 0px 1px 3px 3px #eee -->
</template>

<script>
import SpinnerLoader from "@/components/SpinnerLoader.vue";
import Treeselect from '@riophae/vue-treeselect';
import Swal from 'sweetalert2';
import { ValidationProvider, ValidationObserver, extend } from 'vee-validate';
import { required, email } from 'vee-validate/dist/rules';

extend("required", required);
extend("email", email);
// extend("alpha",alpha);

extend("alpha_spaces", {
  validate: value => /^[A-Za-z\s]+$/.test(value),
  message: "Name may only contain letters and spaces"
});

extend("mobile", {
  validate: value => value.replace(/\D/g,"").length === 10,
  message: "Mobile must be 10 digits"
});

// File required rule
extend("file_required", {
  validate: value => value instanceof File,
  message: "Please upload a file"
});

export default {
  components: { SpinnerLoader, Treeselect, ValidationProvider, ValidationObserver },
  data() {
    return {
      loader: false,
      groups: [
        { name:"", email:"", mobile:"" },
        { name:"", email:"", mobile:"" }
      ],
      form: { birthdate:"", gender:"", languages:[], cities:[], file:null },
      genderOptions:["Male","Female","Other"],
      languageOptions:["Angular","Vuejs","ReactJs"],
      cityOptions:[
        {id: "mumbai", label: "Mumbai"},
        {id: "pune", label: "Pune"},
        {id: "thane", label: "Thane"},
        {id: "raigad", label: "Raigad"},
        {id: "delhi", label: "Delhi"},
        {id: "nashik", label: "Nashik"},
        {id: "nagpur", label: "Nagpur"},
        {id: "ahmedabad", label: "Ahmedabad"},
        {id: "jaipur", label: "Jaipur"},
        {id: "bangalore", label: "Bangalore"},
        {id: "hyderabad", label: "Hyderabad"},
        {id: "chennai", label: "Chennai"},
        {id: "kolkata", label: "Kolkata"},
        {id: "lucknow", label: "Lucknow"},
        {id: "varanasi", label: "Varanasi"}
      ],
      submittedData:[],
      tableFields:["groupData","birthdate","gender","languages","cities","fileName"],
      fileError: "", // store file error here
    };
  },
  methods:{
    addGroup(){ this.groups.push({name:"",email:"",mobile:""}) },
    removeGroup(idx){ if(this.groups.length>2) this.groups.splice(idx,1) },
    formatMobile(idx){
      let num=this.groups[idx].mobile.replace(/\D/g,"").substring(0,10);
      if(num.length>6) num=`(${num.substring(0,3)}) ${num.substring(3,6)}-${num.substring(6)}`;
      else if(num.length>3) num=`(${num.substring(0,3)}) ${num.substring(3)}`;
      else if(num.length>0) num=`(${num}`;
      this.$set(this.groups[idx],"mobile",num);
    },
    validateFile(e) {
      const file = e.target.files[0];
      if (!file) {
        this.form.file = null;
        this.fileError = "Please upload a file";
        return;
      }
      const allowed = ["image/jpeg","application/pdf"];
      if (!allowed.includes(file.type)) {
        this.form.file = null;
        this.fileError = "Only JPEG/PDF allowed!";
      } else {
        this.form.file = file;
        this.fileError = ""; // clear error
      }
    },
    submitForm(){
      this.loader = true; // show loader

      this.$refs.observer.validate().then(valid=>{

        if (!this.form.file) {
          this.fileError = "Please upload a file";
        }

        if (!valid || this.fileError) {
          Swal.fire({ 
            title: "Error!", 
            text: "All fields are mandatory", 
            icon: "error" 
          });
          return;
        }

        const payload={
          groups:this.groups,
          birthdate:this.form.birthdate,
          gender:this.form.gender,
          languages:this.form.languages,
          cities:this.form.cities.map(c=>c.id||c),
          fileName:this.form.file.name
        };

        // I am using a dummy API for testing purposes, so I am not passing a token. 
        // You can uncomment the lines below and provide your token if needed.

        // const token = "YOUR_JWT_TOKEN_HERE"; // Replace with your token


        fetch('https://jsonplaceholder.typicode.com/posts', {
          method: 'POST',
          headers: { 
            'Content-Type': 'application/json',
            // 'Authorization' : `${token}`  // Uncomment and provide token if required
          },
          body: JSON.stringify(payload)
        })
        .then(res => res.json())
        .then(data => {
          console.log("Form submitted:", data);
          this.submittedData.push({
            groupData: JSON.stringify(this.groups),
            birthdate:this.form.birthdate,
            gender:this.form.gender,
            languages:this.form.languages.join(", "),
            cities:this.form.cities.map(c=>c.id||c).join(", "),
            fileName:this.form.file.name
          });

          Swal.fire({
            title:'Form submitted!', 
            icon:'success', 
            confirmButtonText:'OK'
          });

          // Reset form
          this.groups=[{name:"",email:"",mobile:""},{name:"",email:"",mobile:""}];
          this.form={birthdate:"", gender:"", languages:[], cities:[], file:null};
          // Reset all validation errors and upload fil error  
          this.$refs.observer.reset(); // <-- this clears all ValidationProvider errors
          this.fileError = "";
        })
        .catch(error=>{ 
          Swal.fire({
            title:'Error', 
            text:error.message,
            icon:'error'
          }) 
        });
      })
      .finally(() => {
        this.loader = false; // hide loader
      });//end of finally
    }
  }
};
</script>

<style scoped>
.text-danger{ font-size:0.85rem; }
.boxShadow {box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);}
</style>
