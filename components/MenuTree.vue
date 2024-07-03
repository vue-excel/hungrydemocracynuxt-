<template> 
    <v-list :style="indent">
        <div 
            v-for="item in items"
            :key="item.title"
        >
        <v-list-group
            v-model="item.active"
            no-action
            v-if="item.children"
        >
            <template v-slot:activator>
            <v-list-item>
                <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item>
            </template>
            <menu-tree :items="item.children" :depth="depth + 1"></menu-tree>
        </v-list-group>
        <v-list-item v-else>
            <nuxt-link :to="{ path: `/recipe-type/${urlRewrite(item.title)}` }">
            <v-list-item>
                <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item>
            </nuxt-link>
        </v-list-item>
        </div>
    </v-list>
</template>

<script>
export default {
    name: 'menu-tree',
    props: ['items', 'depth'],
    computed: {
      indent() {
        return { transform: `translate(${this.depth * 10}px)` }
      }
    },
    methods: {
        urlRewrite(val) {
            let result = val.split('/').join('&');
            result = result.split(' ').join('-');
            return result;
        }
    }
}
</script>
