<template>
  <v-card class="card-body">
    <v-card-title>
      <h3 class="card-title align-items-start flex-column">
        <span class="card-label font-weight-bolder"
          >Add New Database User</span
        >
      </h3>
      <span
          class="text-muted mt-3 font-weight-bold font-size-sm"
          data-nsfw-filter-status="swf"
          >If you have changed your <strong>root</strong> database password,
          please update your new password inside
          <code>/etc/mysql/my.cnf</code> or adding new user won't work.</span
        >
    </v-card-title>
    <div class="mt-5">
      <b-form id="kt_form_databaseuser">
        <b-form-group label="Name">
          <b-form-input
            name="name"
            placeholder="Only alphanumeric and/or _ are allowed"
            v-model="form.name"
          ></b-form-input>
        </b-form-group>
        <b-form-group label="Password">
          <b-input-group>
            <b-form-input
              type="password"
              name="password"
              v-model="form.password"
              placeholder=""
            ></b-form-input>
            <b-input-group-append>
              <b-button variant="success">Add</b-button>
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
        <b-form-group label="Verify Password">
          <b-form-input
            type="password"
            name="cpassword"
            placeholder=""
          ></b-form-input>
        </b-form-group>
        <button
          type="submit"
          class="btn btn-primary btn-block"
          ref="kt_form_server_submit"
        >
          Add User
        </button>
      </b-form>
    </div>
  </v-card>
</template>

<script>
import formValidation from "@/assets/plugins/formvalidation/dist/es6/core/Core";
import Trigger from "@/assets/plugins/formvalidation/dist/es6/plugins/Trigger";
import Bootstrap from "@/assets/plugins/formvalidation/dist/es6/plugins/Bootstrap";
import SubmitButton from "@/assets/plugins/formvalidation/dist/es6/plugins/SubmitButton";
import KTUtil from "@/assets/js/components/util";

import Swal from "sweetalert2";
import { CREATE_DBUSER } from "@/core/services/store/serverDB.module";

export default {
  data() {
    return {
      form: {
        name: "",
        password: "",
      },
    };
  },
  mounted() {
    const create_form = KTUtil.getById("kt_form_databaseuser");
    this.fv = formValidation(create_form, {
      fields: {
        name: {
          validators: {
            notEmpty: {
              message: "Name is required",
            },
          },
        },
        password: {
          validators: {
            notEmpty: {
              message: "Password is required",
            },
          },
        },
        cpassword: {
          validators: {
            notEmpty: {
              message: "Confirm password is required",
            },
            identical: {
              compare: function () {
                return create_form.querySelector('[name="password"]').value;
              },
              message: "The password and its confirm are not the same",
            },
          },
        },
      },
      plugins: {
        trigger: new Trigger(),
        submitButton: new SubmitButton(),
        bootstrap: new Bootstrap(),
      },
    });
    this.fv.on("core.form.valid", this.createUser);
    this.fv.on("core.form.invalid", () => {});
  },
  methods: {
    createUser() {
      // set spinner to submit button
      const submitButton = this.$refs["kt_form_server_submit"];
      submitButton.classList.add("spinner", "spinner-light", "spinner-right");
      const removeSpinner = () => {
        submitButton.classList.remove(
          "spinner",
          "spinner-light",
          "spinner-right"
        );
      };
      const payload = {
        name: this.form.name,
        password: this.form.password,
        serverId: this.$parent.serverId,
      };
      this.$store
        .dispatch(CREATE_DBUSER, payload)
        .then(() => {
          removeSpinner();
          this.onCreateSuccess(payload.name);
        })
        .catch(() => {
          removeSpinner();
        });
    },
    onCreateSuccess(name) {
      Swal.fire({
        title: "",
        text: "Database user " + name + " has been successfully created",
        icon: "success",
        confirmButtonClass: "btn btn-secondary",
        heightAuto: false,
      }).then(() => {
        console.log();
        this.$router.push({
          name: "server-database",
        });
      });
    },
  },
};
</script>