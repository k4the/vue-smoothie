<template>
  <div class="add-smoothie container">
    <h2 class="center-align indigo-text" v-if="mode === 'add'">Add Smoothie</h2>
    <h2 v-if="smoothie && smoothie.title && mode === 'edit'">Edit {{smoothie.title}}</h2>
    <form @submit.prevent="addEditSmoothie">
      <div class="field title">
        <label for="title">Title</label>
        <input type="text" name="title" v-model="smoothie.title">
      </div>
      <div v-for="(ing, index) in smoothie.ingredients" :key="index" class="field">
        <label for="ingredients">Ingredient</label>
        <input type="text" name="ingredient" v-model="smoothie.ingredients[index]">
        <i class="material-icons delete" @click="deleteIng(ing)">delete</i>
      </div>
      <div class="field add-ingredient">
        <label for="add-ingredient">Add ingredient</label>
        <input type="text" name="add-ingredient" @keydown.tab.prevent="addIng" v-model="another">
      </div>
      <p v-if="feedback" class="red-text">{{feedback}}</p>
      <div class="field center-align">
        <button class="btn pink">Save</button>
      </div>
    </form>
  </div>
</template>
<script>
import db from '@/firebase/init';
import slugify from 'slugify';
export default {
  name: 'AddSmoothie',
  data() {
    return {
      title: null,
      another: null,
      feedback: null,
      slug: null,
      mode: 'add',
      smoothie: {
        title: null,
        slug: null,
        ingredients: []
      }
    };
  },
  methods: {
    addEditSmoothie() {
      if (this.smoothie.title) {
        this.feedback = null;
        this.smoothie.slug = slugify(this.smoothie.title, {
          replacement: '-',
          remove: /[$_+.()'"!\-@]/g,
          lower: true
        });
        if (this.mode === 'add') {
          this.addSmoothie();
        } else {
          this.editSmoothie();
        }
      } else {
        this.feedback = 'You must enter a smoothie title';
      }
    },
    addSmoothie() {
      db.collection('smoothies')
        .add({
          title: this.smoothie.title,
          ingredients: this.smoothie.ingredients,
          slug: this.smoothie.slug
        })
        .then(() => {
          this.$router.push({ name: 'Index' });
        })
        .catch(err => {
          console.log(err);
        });
    },
    editSmoothie() {
      console.log(this.smoothie.ingredients);
      db.collection('smoothies')
        .doc(this.smoothie.id)
        .update({
          title: this.smoothie.title,
          ingredients: this.smoothie.ingredients,
          slug: this.smoothie.slug
        })
        .then(() => {
          this.$router.push({ name: 'Index' });
        })
        .catch(err => {
          console.log(err);
        });
    },
    addIng() {
      if (this.another) {
        this.smoothie.ingredients.push(this.another);
        this.another = null;
        this.feedback = null;
      } else {
        this.feedback = 'You must enter a value';
      }
    },
    deleteIng(ing) {
      this.smoothie.ingredients = this.smoothie.ingredients.filter(
        ingredient => {
          return ingredient !== ing;
        }
      );
    }
  },
  created() {
    if (this.$route.params && this.$route.params.smoothie_slug) {
      this.mode = 'edit';
      console.log('mode', this.mode);
      let ref = db
        .collection('smoothies')
        .where('slug', '==', this.$route.params.smoothie_slug);
      ref.get().then(snapshot => {
        snapshot.forEach(doc => {
          this.smoothie = doc.data();
          this.smoothie.id = doc.id;
        });
      });
    }
  }
};
</script>
<style>
.add-smoothie {
  margin-top: 60px;
  padding: 20px;
  max-width: 500px;
}

.add-smoothie h2 {
  font-size: 2em;
  margin: 20px auto;
}
.add-smoothie .field {
  margin: 20px auto;
  position: relative;
}

.add-smoothie .delete {
  position: absolute;
  right: 0;
  bottom: 16px;
  color: #aaa;
  font-size: 1.4em;
  cursor: pointer;
}
</style>


