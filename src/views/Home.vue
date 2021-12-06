<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Jeux de devinette {{nbrc}} <ion-label v-if="disabled">Timer : {{timer}}</ion-label></ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
       <ion-card>
        <img src="https://cdn2.psychologytoday.com/assets/styles/manual_crop_1_91_1_1528x800/public/field_blog_entry_teaser_image/2020-08/cube-689618_1920.jpg?itok=s6GmdgEZ" />
        <ion-card-header>
          <ion-card-subtitle>Tentative : {{tentative}}</ion-card-subtitle>
          <ion-card-title><ion-input placeholder="Tapez un nombre entre 0 et 100" v-model="nbrp" :disabled="inputdisabled"></ion-input> <ion-button size="small" color="tertiary" @click="valider()" :disabled="inputdisabled">Valider</ion-button><ion-button size="small" color="light" @click="rand()" :disabled="disabled">Nouveau</ion-button></ion-card-title>
        </ion-card-header>
        <ion-card-content>
Au début du jeu, suite à l’appui sur le bouton « Nouveau », on choisit un nombre aléatoire entre 0 et 100. Via un champs de saisi, l’utilisateur propose un premier nombre </ion-card-content>
<ion-button @click="classement()" expand="block">Classement</ion-button>
      </ion-card>
      <ion-card v-if="affclass">
        <ion-card-header>Classement :</ion-card-header>
        <ion-item v-for="(c, index) in clas" :key="index">
          <ion-avatar slot="start">
            <img src="https://www.kindpng.com/picc/m/269-2697881_computer-icons-user-clip-art-transparent-png-icon.png">
          </ion-avatar>
          <ion-label>
            <h2>{{index+1}}</h2>
            <h3>{{c.name}}</h3>
            <p>Score : {{c.score}} Time : {{c.time}}</p>
          </ion-label>
        </ion-item>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonContent, IonHeader, IonPage,IonItem, IonTitle, IonToolbar,IonButton,toastController,alertController,IonInput } from '@ionic/vue';
import { defineComponent } from 'vue';
import axios from 'axios';
export default defineComponent({
  data() {
    return {
      nbrc:0,
      nbrp:0,
      tentative:5,
      i:1,
      disabled:false,
      inputdisabled:true,
      msg:'',
      score:0,
      name:'',
      historique:'',
      clas:[],
      affclass:false,
      timer:0,
      
    }
  },
  methods: {
    rand(){
      this.nbrc=Math.ceil(Math.random()*100);
      this.disabled=true;
      if (this.disabled) {
        this.timer=0;
        setInterval(() => {
        this.timer += 1;
      }, 1000);
      }
      this.inputdisabled=false;
    },
    classement(){
      this.affclass=true;
      axios.get('http://localhost:3000/joueurs?_sort=score&_order=desc').then(response=>this.clas=response.data)
    },
    async valider(){

      if ((this.nbrp>this.nbrc)&&(this.tentative>0)) {
        this.msg='Plus Petit';
        this.historique+=this.i+' : Votre chiffre : '+this.nbrp+' '+this.msg+'<br><br>';
        this.tentative-=1;
        this.i+=1;
        const toast = await toastController
        .create({
          message: this.msg,
          duration: 3000
        })
      return toast.present();
      }
      else if ((this.nbrp<this.nbrc)&&(this.tentative>0)) {
        this.msg='Plus Grand';
        this.historique+=this.i+' : Votre chiffre : '+this.nbrp+' '+this.msg+'<br><br>';
        this.tentative-=1;
        this.i+=1;
        const toast = await toastController
        .create({
          message: this.msg,
          duration: 3000
        })
      return toast.present();
      }
      else if ((this.nbrp==this.nbrc)&&(this.tentative>0)) {
        
        this.score=(this.tentative+1)*10;
        this.historique+=this.i+' : Votre chiffre : '+this.nbrp+' Exact! <br><br>';
        this.msg='Félicitation Votre score est '+this.score+'<br><br>'+this.historique+ 'Tapez Votre Nom :';
        const alert = await alertController.create({
        header: 'Historique',
        message: this.msg,
        inputs: [
        {
          name: 'name',
          type: 'text'
        }],    
         buttons: [
          {
              text: 'Ok',
              handler: (alertData) => { //takes the data 
                this.name=alertData.name;
                axios.post('http://localhost:3000/joueurs',{
                  name:this.name,
                  time:this.timer,
                  score:this.score,
                 })
            }
            }
          ]
      });
      await alert.present();
      
      this.nbrc=0;
      this.nbrp=0;
      this.tentative=5;
      this.disabled=false;
      this.inputdisabled=true;
      }
      else{
        this.msg='Essayez encore!';
        this.nbrc=0;
      this.nbrp=0;
      this.tentative=5;
      this.disabled=false;
      this.inputdisabled=true;
        const toast = await toastController
        .create({
          message: this.msg,
          duration: 3000
        });
      return toast.present();
      
      }
      


    }
    
  },
  name: 'Home',
  components: {
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar,
    IonButton,
    IonInput,IonItem
  }
});
</script>

<style scoped>
#container {
  text-align: center;
  
  position: absolute;
  left: 0;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  
  color: #8c8c8c;
  
  margin: 0;
}

#container a {
  text-decoration: none;
}
</style>