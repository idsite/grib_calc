<template>
  <div class="calc-page">
    <div class="calc">
      <div class="calc-page__items-sub items-sub">
        <div class="items-sub__item" v-for="(item, i) in subs" :key="i">
          <v-text-field
              label="Name"
              v-model="item.name"
          ></v-text-field>
          <v-text-field
              label="Weight"
              v-model="item.weight"
          ></v-text-field>
          <v-text-field
              label="Humidity %"
              v-model="item.hum"
          ></v-text-field>
          <div class="items-sub__item-precent">
            <span v-if="item.weight>0 && totalWeightWithoutHum>0" >{{ Math.round((100/totalWeightWithoutHum)*(item.weight-((item.hum/100)*item.weight)))}}%</span>
          </div>
        </div>
      </div>


      <v-table>
        <template v-slot:default>
          <tbody>
          <tr>
            <th class="text-left">
              Требуемая влажность %
            </th>
            <td class="text-left">
              <v-text-field
                  label="Humidity"
                  v-model="hum_target"
              ></v-text-field>
            </td>
          </tr>
          <tr>
            <th class="text-left">
              Общая масса сухих компонентов
            </th>
            <td class="text-left">
              {{ totalWeight }}
            </td>
          </tr>
          <tr>
            <th class="text-left">
              Масса воды которую нужно добавить
            </th>
            <td class="text-left">
              {{ waterWeight }}
            </td>
          </tr>
          <tr>
            <th class="text-left">
              Масса готового субстрата
            </th>
            <td class="text-left calc__row-weight">
              <div v-if="isEditWeight">
                <v-text-field
                    label="Weight"
                    v-model="editWeight"
                ></v-text-field>
                <v-icon @click="isEditWeight = false">mdi-content-save-check-outline</v-icon>
              </div>
              <div v-else>
                <span>{{ waterWeight+totalWeight }}</span>
                <v-icon  @click="isEditWeight = true"    start
                         icon="mdi-pencil" ></v-icon>
              </div>
            </td>
          </tr>
          </tbody>
        </template>
      </v-table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Calc',
  data() {
    return {
      isEditWeight: false,
      editWeightOld: 0,
      editWeight: 0,
      isInit: false,
      hum_target: 65,
      totalWeight: 0,
      totalWeightWithoutHum: 0,
      waterWeight: 0,
      subs: [
        {
          name: 'Опилки',
          hum: 24,
          weight: 5700
        },
        {
          name: 'Отруби',
          hum: 11,
          weight: 1300
        },
        {
          name: 'Гипс',
          hum: 5,
          weight: 300
        }
      ]
    }
  },
  methods: {
    calc() {
      this.totalWeight = 0;
      this.waterWeight = 0;
      this.totalWeightWithoutHum = 0;
      let totalWeightHum = 0;

      if (this.subs.length && +(this.subs[this.subs.length - 1].weight) > 0) {
        this.subs.push(
            {
              name: '',
              hum: 0,
              weight: 0
            }
        );
      }

      this.subs.forEach((item) => {
        if (item.weight && (+item.weight) > 0) {
          this.totalWeight += (+item.weight);
          let w = (+item.weight);

          if (item.hum && (+item.hum) >= 0) {
            w = w-((item.hum/100)*w)
            totalWeightHum += ((+item.weight) * (+item.hum));
          }
          this.totalWeightWithoutHum+=w;
        }

      });
      if (this.totalWeight === 0 || (+this.hum_target)<=0 || (+this.hum_target)>=100) {
        return;
      }

      this.waterWeight = Math.round( (((+this.hum_target)*this.totalWeight)-totalWeightHum)/(100-this.hum_target));

      if (this.isInit) {
        localStorage.setItem('calc', JSON.stringify({
          subs: this.subs,
          hum_target: this.hum_target
        }));
      }

    }
  },
  watch: {
    isEditWeight(val) {
      if (val) {
        this.editWeight  = this.waterWeight+this.totalWeight;
        this.editWeightOld =  this.editWeight;
      } else {
        if (this.editWeight != this.editWeightOld && +this.editWeight!==0) {
          const p = this.editWeight / this.editWeightOld;
          this.subs = this.subs.map(v=>({
            ...v,
            weight: Math.round(v.weight * p)
          }));
        }
      }
    },
    subs: {
      deep: true,
      immediate: true,
      handler() {
        this.calc();
      }
    },
    hum_target() {
      this.calc();
    }
  },
  created() {
    if (localStorage.getItem('calc')) {
      const calc = JSON.parse(localStorage.getItem('calc'));
      if (calc.subs) {
        this.subs = calc.subs.map(({name,hum,weight})=>({name,hum,weight}));
      }
      if (calc.hum_target) {
        this.hum_target = calc.hum_target;
      }
    }
    this.isInit = true;
  },
  components: {},
}

</script>
<style lang="scss">
.calc-page {
  .calc {
    max-width: 700px;
    &__row-weight {
      &>div {
        display: flex;
        align-items: center;
        &>* {
          margin-right: 10px;
          &:last-child {
            margin-right: 0;
          }
        }
      }
    }
  }

  padding: 30px;
  background-color: white;

  .items-sub {
    &__item {
      margin-bottom: 20px;
      display: grid;
      grid-column-gap: 20px;
      grid-template-columns: auto auto auto;
    }
  }
}
</style>
