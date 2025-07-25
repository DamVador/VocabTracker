<script setup lang="ts">
import AuthenticatedLayout from '@/layouts/AuthenticatedLayout.vue';
import { Head, Link, router } from '@inertiajs/vue3';
import { ref, watch } from 'vue';
import TextInput from '@/components/TextInput.vue';
import PrimaryButton from '@/components/PrimaryButton.vue';
import Pagination from '@/components/Pagination.vue';
import Checkbox from '@/components/Checkbox.vue';

defineOptions({ layout: AuthenticatedLayout });

const props = defineProps({
    words: Object,
    filters: Object,
    allTags: Array,
    allLearningStatuses: Array,
});

const form = ref({
    search_pinyin: props.filters.search_pinyin || '',
    search_translation: props.filters.search_translation || '',
    tag: props.filters.tag || '',
    sort_by: props.filters.sort_by || 'created_at',
    sort_direction: props.filters.sort_direction || 'desc',
    learning_statuses: props.filters.learning_statuses || [],
});

const applyFilters = () => {
    router.get(route('words.index'), form.value, {
        preserveState: true,
        preserveScroll: true,
        replace: true,
    });
};

watch(
    () => form.value,
    () => {
        // Debounce the filter application for better performance
        if (applyFilters.timeout) {
            clearTimeout(applyFilters.timeout);
        }
        applyFilters.timeout = setTimeout(applyFilters, 300);
    },
    { deep: true }
);

const resetFilters = () => {
    form.value = {
        search_pinyin: '',
        search_translation: '',
        tag: '',
        sort_by: 'created_at',
        sort_direction: 'desc',
        learning_statuses: [],
    };
};

const deleteWord = (wordId: number) => {
    if (confirm('Are you sure you want to delete this word from your dictionary? This action cannot be undone.')) {
        router.delete(route('words.destroy', { word: wordId }), {
            preserveState: true,
            preserveScroll: true,
            onSuccess: () => {
                console.log(`Word ${wordId} deleted successfully.`);
            },
            onError: (errors) => {
                console.error("Error deleting word:", errors);
                alert("Failed to delete word. Please try again.");
            }
        });
    }
};
</script>

<template>

    <Head title="My Words" />

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg p-6 text-black">

                <div class="flex flex-col sm:flex-row justify-between items-center gap-4 mb-6">
                    <h2 class="font-semibold text-xl text-gray-800 leading-tight mb-0">My Vocabulary</h2>
                    <Link :href="route('words.create')">
                    <PrimaryButton>Add New Word</PrimaryButton>
                    </Link>
                </div>

                <div class="mb-6 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 xl:grid-cols-6 gap-4 items-end">
                    <div>
                        <label for="search_pinyin" class="block text-sm font-medium text-gray-700 mb-1">Search
                            Pinyin</label>
                        <TextInput id="search_pinyin" type="text" v-model="form.search_pinyin"
                            placeholder="Search by Pinyin..." class="w-full" />
                    </div>

                    <div>
                        <label for="search_translation" class="block text-sm font-medium text-gray-700 mb-1">Search
                            Translation</label>
                        <TextInput id="search_translation" type="text" v-model="form.search_translation"
                            placeholder="Search by translation..." class="w-full" />
                    </div>

                    <div>
                        <label for="tag" class="block text-sm font-medium text-gray-700 mb-1">Filter by Tag</label>
                        <select id="tag" v-model="form.tag"
                            class="block w-full border-gray-300 rounded-md shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                            <option value="">All Tags</option>
                            <option v-for="tagName in allTags" :key="tagName" :value="tagName">
                                {{ tagName.charAt(0).toUpperCase() + tagName.slice(1) }}
                            </option>
                        </select>
                    </div>

                    <div>
                        <label for="sort_by" class="block text-sm font-medium text-gray-700 mb-1">Sort By</label>
                        <select id="sort_by" v-model="form.sort_by"
                            class="block w-full border-gray-300 rounded-md shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                            <option value="pinyin">Pinyin</option>
                            <option value="translation">Translation</option>
                            <option value="failed_attempts">Failed Attempts</option>
                            <option value="last_revision_date">Last Revision</option>
                            <option value="learning_status">Learning Status</option>
                        </select>
                    </div>

                    <div>
                        <label for="sort_direction"
                            class="block text-sm font-medium text-gray-700 mb-1">Direction</label>
                        <select id="sort_direction" v-model="form.sort_direction"
                            class="block w-full border-gray-300 rounded-md shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                            <option value="asc">Ascending</option>
                            <option value="desc">Descending</option>
                        </select>
                    </div>

                    <div class="col-span-full md:col-span-2 lg:col-span-1">
                        <label class="block text-sm font-medium text-gray-700 mb-1">Status Filter</label>
                        <div class="flex flex-wrap gap-2">
                            <div v-for="status in allLearningStatuses" :key="status" class="flex items-center">
                                <Checkbox :id="`status-${status}`" :value="status"
                                    v-model:checked="form.learning_statuses" class="mr-1" />
                                <label :for="`status-${status}`" class="text-sm text-gray-600">{{ status }}</label>
                            </div>
                        </div>
                    </div>

                    <div class="col-span-full md:col-span-2 lg:col-span-1">
                        <button @click="resetFilters"
                            class="inline-flex items-center px-4 py-2 bg-gray-200 border border-transparent rounded-md font-semibold text-xs text-gray-800 uppercase tracking-widest hover:bg-gray-300 focus:bg-gray-300 active:bg-gray-400 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150 w-full">
                            Reset Filters
                        </button>
                    </div>
                </div>

                <div v-if="words.data.length > 0">
                    <div class="overflow-x-auto shadow-md sm:rounded-lg hidden sm:block">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Chinese Word
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Pinyin
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Translation
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Tags
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Added On
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Failed Attempts
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Last Revision
                                    </th>
                                    <th scope="col"
                                        class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                                        Status
                                    </th>
                                    <th scope="col" class="relative px-6 py-3">
                                        <span class="sr-only">Actions</span>
                                    </th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                <tr v-for="word in words.data" :key="word.id">
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                                        {{ word.chinese_word }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
                                        {{ word.pinyin }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        {{ word.translation }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        <span v-for="(tag, index) in word.tags" :key="index"
                                            class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium bg-indigo-100 text-indigo-800 mr-1">
                                            {{ tag }}
                                        </span>
                                        <span v-if="word.tags.length === 0" class="text-gray-400">None</span>
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        {{ word.created_at }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        {{ word.failed_attempts }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        {{ word.last_revision_date }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                                        {{ word.learning_status }}
                                    </td>
                                    <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                                        <Link :href="route('words.edit', { word: word.id })"
                                            class="text-indigo-600 hover:text-indigo-900 mr-4">Edit</Link>
                                        <button @click="deleteWord(word.id)"
                                            class="cursor-pointer text-red-600 hover:text-red-900">Delete</button>
                                    </td>
                                </tr>
                            </tbody>
                        </table>

                    </div>

                    <ul class="block sm:hidden mt-4 bg-white shadow-sm rounded-lg divide-y divide-gray-200">
                        <li v-for="word in words.data" :key="word.id" class="p-4 flex flex-col">
                            <div class="flex justify-between items-start mb-2">
                                <div class="flex-1">
                                    <p class="text-lg font-medium text-gray-900">{{ word.chinese_word }} ({{ word.pinyin
                                        }})</p>
                                    <p class="text-base text-gray-600">{{ word.translation }}</p>
                                </div>
                                <div class="flex flex-col items-end gap-1 ml-4">
                                    <span v-if="word.learning_status"
                                        class="px-2.5 py-0.5 bg-blue-100 text-blue-800 rounded-full text-xs font-medium">
                                        Status: {{ word.learning_status }}
                                    </span>
                                    <span v-if="word.failed_attempts > 0"
                                        class="px-2.5 py-0.5 bg-red-100 text-red-800 rounded-full text-xs font-medium">
                                        Failures: {{ word.failed_attempts }}
                                    </span>
                                    <span v-if="word.last_revision_date"
                                        class="px-2.5 py-0.5 bg-gray-100 text-gray-800 rounded-full text-xs font-medium">
                                        Last Rev: {{ word.last_revision_date }}
                                    </span>
                                </div>
                            </div>
                            <div class="flex flex-wrap gap-1 mb-2">
                                <span v-for="tag in word.tags" :key="tag"
                                    class="px-2 py-0.5 bg-gray-200 rounded-full text-xs font-medium text-gray-700">
                                    {{ tag }}
                                </span>
                                <span v-if="word.tags.length === 0" class="text-gray-400 text-xs italic">No tags</span>
                            </div>
                            <div class="flex justify-end gap-3 mt-2">
                                <Link :href="route('words.edit', { word: word.id })"
                                    class="text-indigo-600 hover:text-indigo-900 text-sm font-medium">Edit</Link>
                                <button @click="deleteWord(word.id)"
                                    class="text-red-600 hover:text-red-900 text-sm font-medium">Delete</button>
                            </div>
                        </li>
                    </ul>

                    <Pagination :pagination="words" :current-filters="form" />
                </div>
                <div v-else class="text-center text-gray-500 py-8">
                    No words found matching your criteria.
                    <p class="mt-2">Try adjusting your filters or
                        <Link :href="route('words.create')" class="text-indigo-600 hover:underline">add a new word
                        </Link>.
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>