<script setup>
import { ref, onMounted } from "vue";
import axios from 'axios';
import algoliasearch from 'algoliasearch';
import 'instantsearch.css/themes/satellite-min.css';

const results = ref([]);
const url = 'https://jsonplaceholder.typicode.com/users';
const searchClient = algoliasearch(
  'FUT2K0OJPA',
  '2c1f0e7579969d2d27925b9ea59dd41e'
);
const index = searchClient.initIndex('user_search');

const fetchUsers = () => {
  axios.get(url)
    .then(response => {
      response.data.forEach((element, index) => {
        // Set the objectId as the index for each element returned
        const objectID = `userId${index}`
        // Add the objectId to the returned element object
        element = { ...element, objectID }
        // Add the element object to the results array
        results.value.push(element)
      });
    })
    // Set attributes to be highlighted
    .then(() => {
      index.setSettings({
        attributesToHighlight: [
          'name',
          'email',
          'address.street',
          'address.city'
        ]
      })
    })
    // Send response data to Algolia index and store as records
    .then(() => {
      index.saveObjects(results.value)
    })
    .catch(error => {
      console.log(error);
    })
}

onMounted(() => {
  fetchUsers();
})
</script>


<template>
  <ais-instant-search :search-client="searchClient" index-name="user_search" class="w-11/12 md:w-9/12 xl:w-1/2 mx-auto">
    <ais-search-box class="my-10" />
    <ais-hits>
      <template v-slot="{ items }">
        <div v-for="item in items" class="user-card mb-8">
          <div class="inner-card-wrapper px-8 py-4 md:flex justify-between shadow-md">
            <div class="user-info md:ml-20">
              <h2 class="mb-2 text-xl font-medium">
                <ais-highlight :hit="item" attribute="name" />
              </h2>
              <p>
                <ais-highlight attribute="address.street" :hit="item" />
                <span>
                  ,
                  <ais-highlight attribute="address.city" :hit="item" />
                </span>
              </p>
              <a
                :href="`mailto:${item.email}`"
                class="underline text-green-500 hover:text-green-300"
              >
                <ais-highlight attribute="email" :hit="item" />
              </a>
              <p
                class="mt-2"
              >Lorem ipsum dolor sit amet, consectetur adipisicing elit. Pariatur culpa accusamus eius nisi quam a minus saepe sint veritatis error.</p>
            </div>
          </div>
        </div>
      </template>
    </ais-hits>
  </ais-instant-search>
</template>