<style scoped lang="scss">

  .avatar {
    display: block;
    margin-left: auto;
    margin-right: auto;

    height: 60px;
    width: 60px;
    border-radius: 100%;
    border: solid 1px #dadada;
  }

  .avatar-loading {
    background-color: #dadada;

    i {
      padding: 23px;
    }
  }

  .input-file {
    display: none;
  }

</style>


<template lang="pug">
  .mt-2.container
    .card
      form.mb-0(ref="form", method="patch", @submit.prevent="submit")
        template(v-if="uploadPercentage > 0 && uploadPercentage < 100")
          .avatar.avatar-loading
            i.fa.fa-spin.fa-spinner
        template(v-else)
          img.avatar(:src="avatarPath", @click="$refs.inputFile.click()")

        input.input-file(type="file", name="user[avatar]" ref="inputFile", @change="onFileChange($event.target.name, $event.target.files)")

        hr.mt-1.mb-1

        label
          | {{ $t('models.user.attributes.name') }}
        div.mb-1
          | {{ user.name }}

        label
          | {{ $t('models.user.attributes.email') }}
        div.mb-1
          | {{ user.username }}

        input-field(
          type="password",
          v-model="user.password",
          name="user[password]",
          :error="errors.password"
        )

        input-field(
          type="password",
          v-model="user.password_confirmation",
          name="user[password_confirmation]",
          :error="errors.password_confirmation"
        )

        .text-center
          button.mb-0.button-primary(
            type="submit",
            :disabled="submitting"
          )
            | {{ submitText }}
</template>


<script>
  export default {
    name: 'editProfile',
    data () {
      return {
        i18nScope: 'profile.edit',

        errors: {},

        submitting: false,
        user: null,
        params: null,
        uploadPercentage: null,
        files: [],
        preview: null
      }
    },

    computed: {
      avatarPath() {
        if(this.preview) return this.preview
        return app.secrets.api.host + this.user.avatar.url
      },


      submitText() {
        if (this.submitting) return this.$t('.button.submitting')
        return this.$t('.button.submit')
      },
    },

    methods: {
      loadUser() {
        this.user = this.$auth.user
      },

      onFileChange(fieldName, file) {
        let imageFile = file[0]
        let imageURL = URL.createObjectURL(imageFile)

        this.preview = imageURL
      },

      submit() {
        const formData = new FormData(this.$refs.form)
        this.submitting = true

        this.$http.patch('/cooperative/profile', formData, {
          onUploadProgress: uploadEvent => {
            this.uploadPercentage = uploadEvent.loaded / uploadEvent.total * 100

            console.log(this.uploadPercentage)
          }
        })
          .then((response) => {
            this.$notifications.clear()
            this.$notifications.info(this.$t('.notifications.success'))
            this.errors = {}

            this.$auth.setUser(response.data.user)

            this.loadUser()
          })
          .catch((err) => {
            let errors = _.dig(err, 'response', 'data', 'errors') || {}

            this.$notifications.error(this.$t('.notifications.failure'))
            this.errors = this.$i18n.errify(errors, { model: 'user' })
          })
          .then(() => {
            this.submitting = false
          })
      },
    },

    created: function () {
      this.loadUser()
      this.$emit('navbarTitleChanged', this.$t('title'))
    }
  }

</script>
