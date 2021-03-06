<template>
  <Popup>
    <template slot="header">
      <div>
        <h5>
          <span class="cap-first">
            {{ titreNom }} | Rapport trimestriel d'activité
          </span>
        </h5>
        <h2 class="cap-first mb-0">
          {{ trimestreNom }} {{ rapport.contenu.annee }}
        </h2>
      </div>
    </template>

    <div v-if="editable">
      <div class="p-s bg-info color-bg mb">
        Besoin d'aide pour remplir ce rapport ? Appelez le 06.61.26.42.89
      </div>

      <div
        v-for="contenu in contenus"
        :key="contenu.id"
      >
        <div class="tablet-blobs">
          <div class="tablet-blob-1-3 tablet-pt-s pb-s">
            <h6>{{ contenu.nom }}</h6>
          </div>
          <div class="mb tablet-blob-2-3">
            <input
              v-model.number="rapport.contenu[contenu.id]"
              type="number"
              class="p-s mb-s"
              placeholder="…"
            >
            <!-- eslint-disable vue/no-v-html -->
            <p
              class="h5 mb-0"
              v-html="contenu.description"
            />
          </div>
        </div>

        <hr>
      </div>


      <h4>Statut des travaux</h4>
      <div
        v-for="mois in rapport.contenu.travaux"
        :key="mois.id"
      >
        <hr>
        <div class="tablet-blobs">
          <div class="mb tablet-blob-1-3 tablet-pt-s pb-s">
            <h6>{{ mois.nom }}</h6>
          </div>
          <div class="mb tablet-blob-2-3">
            <label
              v-for="statut in travauxStatuts"
              :key="statut.id"
            >
              <input
                v-model="mois[statut.id]"
                type="checkbox"
              >{{ statut.nom }}
            </label>
          </div>
        </div>
      </div>

      <hr>
      <h4>Informations complémentaires</h4>
      <hr>
      <div class="mb">
        <textarea
          v-model="rapport.contenu.complement"
          class="p-s mb-s"
        />
        <p class="h5 mb-0">
          Toute information sur les événements marquants du trimestre (accident,
          incident, arrêt ou suspension d’activité en précisant les raisons,
          évolution de l’exploitation, difficultés rencontrées, etc.).
        </p>
      </div>
    </div>

    <div v-else>
      <div
        v-for="contenu in contenus"
        :key="contenu.id"
      >
        <div class="tablet-blobs">
          <div class="tablet-blob-1-3 pb-s">
            <h6>{{ contenu.nom }}</h6>
          </div>
          <div class="tablet-blob-2-3">
            <p :class="{'color-warning': !(rapport.contenu[contenu.id] || rapport.contenu[contenu.id] === 0) }">
              {{ rapport.contenu[contenu.id] || rapport.contenu[contenu.id] === 0 ? rapport.contenu[contenu.id] : 'À compléter pour valider' }}
            </p>
          </div>
        </div>

        <hr>
      </div>

      <h4>Statut des travaux</h4>
      <div
        v-for="mois in rapport.contenu.travaux"
        :key="mois.id"
      >
        <hr>
        <div class="tablet-blobs">
          <div class="tablet-blob-1-3 pb-s">
            <h6>{{ mois.nom }}</h6>
          </div>
          <div class="tablet-blob-2-3">
            <p :class="{'color-warning': !travauxStatuts.filter(ts => mois[ts.id]).length}">
              {{ travauxStatuts.filter(ts => mois[ts.id]).map(ts => ts.nom).join(', ') || 'À compléter pour valider' }}
            </p>
          </div>
        </div>
      </div>

      <hr>
      <div v-if="rapport.contenu.complement">
        <h4>Informations complémentaires</h4>
        <hr>
        <div class="mb">
          <p>{{ rapport.contenu.complement }}</p>
        </div>
      </div>

      <div
        class="p-s bg-warning color-bg bold mb"
        :class="{ hide: !complete }"
      >
        Une fois validé ce formulaire ne sera plus modifiable.
      </div>
    </div>

    <template slot="footer">
      <Messages :messages="messages" />

      <div
        v-if="editable"
        class="tablet-blobs"
      >
        <div class="mb tablet-mb-0 tablet-blob-1-3">
          <button
            class="btn-border rnd-xs p-s full-x"
            @click="cancel"
          >
            Annuler
          </button>
        </div>
        <div
          class="tablet-blob-2-3"
        >
          <button
            class="btn-flash rnd-xs p-s full-x"
            @click="preview"
          >
            Prévisualiser
          </button>
        </div>
      </div>
      <div
        v-else
        class="tablet-blobs"
      >
        <div class="mb tablet-blob-1-3 tablet-mb-0">
          <button
            class="btn-border rnd-xs p-s full-x"
            @click="edit"
          >
            Modifier
          </button>
        </div>
        <div
          class="mb tablet-blob-1-3 tablet-mb-0"
        >
          <button
            class="rnd-xs p-s full-x"
            :class="{ 'btn-flash': !complete, 'btn-border': complete }"
            @click="save(false)"
          >
            Enregistrer
          </button>
        </div>
        <div
          class="tablet-blob-1-3"
          :class="{ disabled: !complete }"
        >
          <button
            class="btn-flash rnd-xs p-s full-x"
            @click="complete && save(true)"
          >
            Valider
          </button>
        </div>
      </div>
    </template>
  </Popup>
</template>

<script>
import Popup from '../ui/popup.vue'
import Messages from '../ui/messages.vue'

export default {
  name: 'CaminoTitreTravauxRapportEditPopup',

  components: {
    Popup,
    Messages
  },

  props: {
    rapport: {
      type: Object,
      default: () => ({})
    },
    trimestreNom: {
      type: String,
      default: ''
    },
    titreNom: {
      type: String,
      default: ''
    }
  },

  data() {
    return {
      editable: true
    }
  },

  computed: {
    messages() {
      return this.$store.state.popup.messages
    },
    entreprises() {
      return this.$store.state.entreprises.list
    },
    substances() {
      return this.$store.state.substances.list
    },
    travauxStatuts() {
      return this.$store.state.titreTravaux.rapportStatuts
    },
    contenus() {
      return this.$store.state.titreTravaux.rapportContenus.filter(
        c =>
          !c.trimestres ||
          c.trimestres.find(tr => tr === this.rapport.contenu.trimestre)
      )
    },
    travauxComplete() {
      return this.rapport.contenu.travaux.reduce(
        (res, mois) =>
          res &&
          (mois.nonDebutes ||
            mois.exploitationEnCours ||
            mois.arretTemporaire ||
            mois.rehabilitation ||
            mois.arretDefinitif),
        true
      )
    },
    contenuComplete() {
      return this.contenus.reduce(
        (res, c) =>
          res &&
          (this.rapport.contenu[c.id] || this.rapport.contenu[c.id] === 0),
        true
      )
    },
    complete() {
      return this.contenuComplete && this.travauxComplete
    }
  },

  created() {
    document.addEventListener('keyup', this.keyup)
  },

  beforeDestroy() {
    document.removeEventListener('keyup', this.keyup)
  },

  methods: {
    preview() {
      this.editable = false
    },

    edit() {
      this.editable = true
    },

    save(confirmation) {
      this.rapport.date = new Date()
      if (confirmation && this.complete) {
        this.rapport.confirmation = confirmation

        this.$store.dispatch('titreTravaux/rapportUpdate', this.rapport)
      } else {
        this.rapport.confirmation = false

        this.$store.dispatch('titreTravaux/rapportUpdate', this.rapport)
      }
    },

    cancel() {
      this.errorsRemove()
      this.$store.commit('popupClose')
    },

    keyup(e) {
      if ((e.which || e.keyCode) === 27) {
        if (this.editable) {
          this.cancel()
        } else {
          this.edit()
        }
      } else if ((e.which || e.keyCode) === 13) {
        if (this.editable) {
          this.preview()
        } else {
          if (this.complete) {
            this.save(true)
          } else {
            this.save(false)
          }
        }
      }
    },

    errorsRemove() {
      this.$store.commit('popupMessagesRemove')
    }
  }
}
</script>
