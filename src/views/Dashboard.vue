<template>
  <div class="dashboard">
    <div class="container">
      <nav class="level">
        <!-- Left side -->
        <div class="level-left">
          <div class="level-item">
            <h1 class="title is-4">Seu Gerenciador de Contatos</h1>
          </div>
        </div>

        <!-- Right side -->
        <div class="level-right">
          <div class="level-item">
            <b-button
              class="is-success"
              label="+"
              @click="showContactAddModal = true"
              data-testId="add_contact_button"
            />
          </div>
          <div class="level-item">
            <div class="field has-addons">
              <p class="control">
                <input
                  class="input"
                  type="text"
                  v-model="searchInput"
                  placeholder="Número do Whats"
                />
              </p>
              <p class="control">
                <button class="button is-primary" @click="search">
                  Buscar
                </button>
              </p>
            </div>
          </div>
        </div>
      </nav>

      <div id="loader" v-if="isLoading === true">
        <img src="../assets/loading.gif" alt="loader" />
      </div>

      <div class="notification is-danger" v-if="contactList.length === 0">
        Contato não cadastrado :(
      </div>

      <div
        class="columns is-multiline"
        data-testId="contact_card"
        v-if="isLoading === false"
      >
        <div
          class="column is-4"
          v-for="contact in contactList"
          :key="contact._id"
        >
          <div class="card">
            <div class="card-content">
              <div class="media">
                <div class="media-left">
                  <figure class="image is-48x48">
                    <img src="../assets/whatsapp.svg" alt="Logo WhatasApp" />
                  </figure>
                </div>
                <div class="media-content">
                  <p
                    class="title is-4"
                    :data-testId="`contact_card_name_${contact.name}`"
                  >
                    {{ contact.name }}
                  </p>
                  <p
                    class="subtitle is-6"
                    :data-testId="`contact_card_number_${contact.number}`"
                  >
                    {{ contact.number }}
                  </p>
                </div>
              </div>

              <div
                class="content"
                :data-testId="`contact_card_description_${contact.description}`"
              >
                {{ contact.description }}
              </div>
            </div>
            <footer class="card-footer">
              <a :href="zaplink(contact.number)" class="card-footer-item">Conversar</a>
              <a href="#" class="card-footer-item btn-remove" @click="remove(contact._id)">Apagar</a>
            </footer>
          </div>
        </div>
      </div>

      <b-modal
        v-model="showContactAddModal"
        has-modal-card
        trap-focus
        :destroy-on-hide="false"
        aria-role="dialog"
        aria-label="Example Modal"
        aria-modal
      >
        <form action="">
          <div class="modal-card" style="width: 450px">
            <header class="modal-card-head">
              <p class="modal-card-title" data-testId="new_contact_form">
                Novo Contato
              </p>
              <button
                type="button"
                class="delete"
                @click="showContactAddModal = false"
              />
            </header>
            <section class="modal-card-body">
              <div class="field">
                <input
                  type="text"
                  class="input is-primary"
                  v-model="form.name"
                  placeholder="Nome Completo"
                  data-testId="form_name"
                />
                <small
                  data-testId="name_error"
                  class="has-text-danger"
                  v-if="errorName === true"
                  >Nome é Obrigatório</small
                >
              </div>
              <div class="field">
                <input
                  type="text"
                  class="input is-primary"
                  v-model="form.number"
                  placeholder="WhatsApp"
                  data-testId="form_number"
                />
                <small
                  data-testId="number_error"
                  class="has-text-danger"
                  v-if="errorNumber === true"
                  >Number é Obrigatório</small
                >
              </div>
              <div class="field">
                <textarea
                  type="textarea"
                  class="input is-primary"
                  v-model="form.description"
                  placeholder="Assunto"
                  data-testId="form_description"
                ></textarea>
                <small
                  data-testId="description_error"
                  class="has-text-danger"
                  v-if="errorDescription === true"
                  >Assunto é Obrigatório</small
                >
              </div>
            </section>
            <footer class="modal-card-foot">
              <b-button
                label="Cadastrar"
                type="button"
                class="button is-success"
                @click="create"
                data-testId="form_btn_cadastrar"
              />
            </footer>
          </div>
        </form>
      </b-modal>
    </div>
  </div>
</template>

<script>
export default {
  name: "Dashboard",
  data() {
    return {
      isLoading: false,
      contactList: [],
      showContactAddModal: false,
      errorName: false,
      errorNumber: false,
      errorDescription: false,
      searchInput: "",
      form: {
        name: "",
        number: "",
        description: "",
      },
    };
  },
  methods: {
    zaplink(number) {
      return `https://api.whatsapp.com/send?phone=55${number}`
    },
    search() {
      this.isLoading == true;
      if (this.searchInput != "") {
        this.contactList = this.contactList.filter(
          (contact) => contact.number === this.searchInput
        );
        this.isLoading == false;
      } else {
        this.list();
      }
    },
    create() {
      this.errorName = false;
      this.errorNumber = false;
      this.errorDescription = false;

      if (this.form.name === "") {
        this.errorName = true;
      }

      if (this.form.number === "") {
        this.errorNumber = true;
      }

      if (this.form.description === "") {
        this.errorDescription = true;
      }

      if (
        this.errorName === false &&
        this.errorNumber === false &&
        this.errorDescription === false
      ) {
        window.axios.post("/contacts", this.form).then(async (res) => {
          await res.data;
          this.form.name = "";
          this.form.number = "";
          this.form.description = "";
          this.showContactAddModal = false;
          this.list();
        });
      }
    },

    remove(contactId) {
      window.axios.delete('/contacts/' + contactId).then(async (res) => {
        await res.data
        this.list()
      })

    },

    list() {
      this.isLoading == true;
      window.axios.get("/contacts").then(async (res) => {
        this.contactList = await res.data;
        this.isLoading == false;
      });
    },
  },
  mounted() {
    this.list();
  },
};
</script>
