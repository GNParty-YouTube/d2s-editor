<template>
  <div>
    <div class="form-row">
      <div class="col-md-12">
        <button type="button" class="btn btn-primary" @click="onEvent('share')">Share</button>
        <button type="button" class="btn btn-primary" @click="onEvent('copy')">Copy</button>
        <span v-if="item.location_id != 6">
          <button type="button" class="btn btn-danger" @click="onEvent('delete')">Delete</button>
        </span>
      </div>
    </div>

    <div class="form-row">
      <div>
        <Item :item.sync="item" clazz="item-edit"></Item>
      </div>

      <ul className="ItemOptions">
        <div class="settings">
          <!-- Base -->
          <label>Base</label>
          <div>
            <select class="edit-box" v-model="item.type" @change="onEvent('update')">
              <option v-for="s in getBases(item.type)" :value="s[0]" :key="s[0]">{{ s[1].n }}</option>
            </select>
          </div>

          <!-- Defense -->
          <template v-if="item.defense_rating">
            <label>&#187;&#187; Defense</label>
            <input class="edit-box" type="number" v-model.number="item.defense_rating" @input="onEvent('update')" min="1" max="1000" disabled/>
          </template>

          <!-- iLevel -->
          <label>Item Level</label>
          <input class="edit-box" type="number" v-model.number="item.level" @input="onEvent('update')" min="1" max="99">

          <!-- Skin -->
          <template  v-if="countSkinsChoices()">
            <label>Skin</label>
            <div>
              <select class="edit-box" v-model.number="item.picture_id" @change="onEvent('update')">
                <option v-for="n in countSkinsChoices()" :value="n - 1" :key="n">{{ skin_names[item.type] ? skin_names[item.type][n-1] : `Skin ${n}` }}</option>
              </select>
            </div>
          </template >

          <template v-if="!item.simple_item">
            <!-- Quality -->
            <label>Quality</label>
            <div>
              <select class="edit-box" v-model.number="item.quality" @change="onEvent('update')">
                <option v-for="rarity in rarities" :value="rarity.key" :key="rarity.key">{{ rarity.value }}</option>
              </select>
            </div>

            <!-- Magic prefix & suffix -->
            <template v-if="item.quality == 4">
              <label>&#187;&#187; Prefix</label>
              <div>
                <select class="edit-box" v-model.number="item.magic_prefix" @change="onEvent('update')">
                  <option value="0">None</option>
                  <option v-for="s in prefixes" :value="s.i" :key="s.i">{{ s.i }} - {{ s.v.n }}</option>
                </select>
              </div>
              <label>&#187;&#187; Suffix</label>
              <div>
                <select class="edit-box" v-model.number="item.magic_suffix" @change="onEvent('update')">
                  <option value="0">None</option>
                  <option v-for="s in suffixes" :value="s.i" :key="s.i">{{ s.i }} - {{ s.v.n }}</option>
                </select>
              </div>
            </template>

            <!-- Rare prefix & suffix -->
            <template v-if="item.quality == 6 || item.quality == 8">
              <label>&#187;&#187; Prefix</label>
              <div>
                <select class="edit-box"  v-model.number="item.rare_name_id" @change="onEvent('update')">
                  <option v-for="s in rare_names" :value="s.i" :key="s.i">{{ s.i }} - {{ s.v.n }}</option>
                </select>
              </div>
              <label>&#187;&#187; Suffix</label>
              <div>
                <select class="edit-box"  v-model.number="item.rare_name_id2" @change="onEvent('update')">
                  <option v-for="s in rare_names" :value="s.i" :key="s.i">{{ s.i }} - {{ s.v.n }}</option>
                </select>
              </div>
            </template>

            <!-- Set name -->
            <template v-if="item.quality == 5">
              <label>&#187;&#187; Set item ID</label>
              <select class="edit-box" v-model.number="item.set_id" @change="onEvent('update')">
                <option v-for="s in set_items" :value="s.id" :key="s.id">{{ s.n }}</option>
              </select>
            </template>

            <!-- Unique name -->
            <template v-if="item.quality == 7">
              <label>&#187;&#187; Unique item ID</label>
              <select class="edit-box" v-model.number="item.unique_id" @change="onEvent('update')">
                <option v-for="s in unq_items" :value="s.id" :key="s.id">{{ s.n }} - {{ s.id }}</option>
              </select>

              <!-- <label>Personalized Name</label>
              <input class="edit-box" type="text" v-model="item.personalized_name" @change="onEvent('update')" pattern="^[A-Za-z]{0,15}$" maxlength="16"/> -->
            </template>
            
            <!-- Ethereal -->
            <label>Ethereal</label>
            <label class="form-check-label"><input class="form-check-input" type="checkbox"
                v-model.number="item.ethereal" :true-value="1" :false-value="0" @change="onEvent('update')"></label>

            <!-- Sockets -->
            <label>Sockets</label>
            <input class="edit-box" type="number" v-model.number="item.total_nr_of_sockets" @input="onEvent('update')" min="0" max="8">
          </template>
        </div>
      </ul>
    </div>

    <span v-if="!item.simple_item">
      <div v-if="item.magic_attributes">
        <div>Item Stats</div>
        <ItemStatsEditor :item-stats.sync="item.magic_attributes" :id="id + 'Magic'" @stat-change="onEvent('update')"></ItemStatsEditor>
      </div>
      <div v-if="item.runeword_attributes">
        <div>Runeword Stats</div>
        <ItemStatsEditor :item-stats.sync="item.runeword_attributes" :id="id + 'Runeword'" @stat-change="onEvent('update')"></ItemStatsEditor>
      </div>
      <div v-if="item.set_attributes">
        <div v-for="(set_attribute, index) in item.set_attributes">
          <div>Set Stats {{index}}</div>
          <ItemStatsEditor :item-stats.sync="set_attribute" :id="id + 'Set' + index" @stat-change="onEvent('update')"></ItemStatsEditor>
        </div>
      </div>
      <div v-if="item.socketed_items">
        <div>Sockets Stats</div>
        <ItemStatsEditor :item-stats.sync="item.socketed_attributes" :id="id + 'Socketed stats'" @stat-change="onEvent('update')"></ItemStatsEditor>
      </div>
      <div v-if="item.socketed_items">
        <div v-for="(socketed_item, index) in item.socketed_items">
          <ItemEditor ref="itemEditor" :item.sync="socketed_item" :id="id + 'Socketed' + index" @item-event="onChildEvent" :location="{ location: 6 }"></ItemEditor>
        </div>
      </div>
    </span>
  </div>
</template>

<script>
import Item from './Item.vue';
import ItemStatsEditor from './ItemStatsEditor.vue';
import utils from '../../utils.js';

export default {
  name: 'ItemEditor',
  props: {
    id: String,
    item: Object,
    location: Object,
  },
  components: {
    Item,
    ItemStatsEditor,
  },
  data() {
    return {
      rarities: [{ key: 1, value: 'Low' }, { key: 2, value: 'Normal' }, { key: 3, value: 'Superior' }, { key: 4, value: 'Magic' }, { key: 5, value: 'Set' }, { key: 6, value: 'Rare' }, { key: 7, value: 'Unique' }, { key: 8, value: 'Crafted' }],
      locations: [{ key: 0, value: 'Stored' }, { key: 1, value: 'Equipped' }, { key: 4, value: 'Cursor' }],
      equipped_locations: [{ key: 1, value: 'Head' }, { key: 2, value: 'Neck' }, { key: 3, value: 'Torso' }, { key: 4, value: 'Right Hand' }, { key: 5, value: 'Left Hand' }, { key: 6, value: 'Right Finger' }, { key: 7, value: 'Left Finger' }, { key: 8, value: 'Waist' }, { key: 9, value: 'Boots' }, { key: 10, value: 'Gloves' }, { key: 11, value: 'Alternate Right Hand' }, { key: 12, value: 'Alternate Left Hand' }],
      storage_pages: [{ key: 1, value: 'Inventory' }, { key: 4, value: 'Cube' }, { key: 5, value: 'Stash' }],
      prefixes: window[`${window.work_mod}_constants_${window.work_version}`].magic_prefixes.map((e,i)=> { return { i:i, v:e }}).filter(e => e.v != null && e.v.n != null),
      suffixes: window[`${window.work_mod}_constants_${window.work_version}`].magic_suffixes.map((e,i)=> { return { i:i, v:e }}).filter(e => e.v != null && e.v.n != null),
      rare_names: window[`${window.work_mod}_constants_${window.work_version}`].rare_names.map((e,i)=> { return { i:i, v:e }}).filter(e => e.v != null && e.v.n != null),
      unq_items: window[`${window.work_mod}_constants_${window.work_version}`].unq_items.filter(item => item.n != null),
      set_items: window[`${window.work_mod}_constants_${window.work_version}`].set_items.filter(item => item.n != null),
      armor_items: Object.entries(window[`${window.work_mod}_constants_${window.work_version}`].armor_items).filter(e => e[1].n != null),
      weapon_items: Object.entries(window[`${window.work_mod}_constants_${window.work_version}`].weapon_items).filter(e => e[1].n != null),
      other_items: Object.entries(window[`${window.work_mod}_constants_${window.work_version}`].other_items).filter(e => e[1].n != null),
      skin_names: {
        "amu": [ "Dot", "Sun", "Penta" ],
        "rin": [ "Coral", "Small Blue", "Big Blue", "Orange", "Crown" ],
        "cm1": [ "Brown", "Bear-foot", "M-skin" ],
        "cm2": [ "Paw", "Horn", "Tower" ],
        "cm3": [ "Eye", "Spaghetti/DNA", "Dragon/Monster" ],
        "jew": [ "Pink", "Blue", "Orange", "Green", "Red", "White" ]
      }
    };
  },
  methods: {
    onUpdate(variable, value) {
      const path = variable.split('.');
      path.shift(); // Should be "item"
      const edited = path.reduceRight(
        (accumulator, currentValue) => ({[currentValue]: accumulator}),
        value
      );
      const newItem = Object.assign(this.item, edited);
      this.$emit('item-event', { item: newItem, type: 'update' });
    },
    onEvent(type, variable, value) {
      this.$emit('item-event', { item: this.item, type: type });
    },
    onChildEvent(e) {
      this.$emit('item-event', { item: e.item, type: e.type });
    },
    // onMove() {
    //   this.$emit('item-event', { item: this.item, location: this.location, type: 'move' });
    // },
    countSkinsChoices() {
      const constants = window[`${window.work_mod}_constants_${window.work_version}`]
      const details = utils.getItemDetails(this.item);
      if (this.item.unique_id) {
        if (details.ui) return 0;
        const unq = constants.unq_items[this.item.unique_id];
        if (unq) {
          if (unq.i) return 0;
          if (unq.hdi) return 0;
        }
      }
      if (this.item.set_id) {
        if (details.si) return 0;
        const set = constants.set_items[this.item.set_id];
        if (set) {
          if (set.i) return 0;
          if (set.hdi) return 0;
        }
      }
      const gfx_list = details.ig || details.hdig;
      return gfx_list ? gfx_list.length : 0;
    },
    findBasesInConstants(code, items) {
      let bases = [];
      const base = items[code];
      if (base) {
        //NORMAL SET UNIQUE CRAFTED
        if (this.item.quality == 5 || this.item.quality == 6 || this.item.quality == 7 || this.item.quality == 8) {
          bases = [base.nc, base.exc, base.elc].filter(id => items[id]);
          //items.filter(e => e[1].nc == code || e[1].exc == code || e[1].elc == code)
        }
        else {
          bases = Object.keys(items).filter(id => {
            const item = items[id];
            if (this.item.given_runeword == 1 && item.gemsockets < this.item.total_nr_of_sockets) return false;
            if (base.c.length > 2) 
              return item.eq1n == base.eq1n 
            else 
              return item.type === base.type
          }).sort((a, b) => items[a].level < items[b].level);
        }
        bases = Object.entries(items).filter(item => bases.includes(item[0]));
      }
      return bases
    },
    getBases(code) {
      if (this.item.type_id == 3) {
        return this.findBasesInConstants(code, window[`${window.work_mod}_constants_${window.work_version}`].weapon_items);
      } else if (this.item.type_id == 1) {
        return this.findBasesInConstants(code, window[`${window.work_mod}_constants_${window.work_version}`].armor_items);
      } else if (this.item.type_id == 4) {
        return this.other_items
      } else {
        return []
      }
    },
  }
};  
</script>