<script setup lang="ts">
import { Link, useForm, usePage } from '@inertiajs/vue3';
import { ref, computed } from 'vue';
import InputError from '@/components/InputError.vue';
import Input from '@/components/Input.vue';
import Select from '@/components/Select.vue';

const props = defineProps({
    user: Object, // User object from Profile/Edit.vue
    countries: Array,
});

const form = useForm({
    username: props.user.username,
    first_name: props.user.first_name,
    last_name: props.user.last_name,
    email: props.user.email,
    country: props.user.country,
    city: props.user.city,
    native_language: props.user.native_language,
    chinese_level: props.user.chinese_level,
});

const verificationLinkSent = ref(false);
const authUser = usePage().props.auth.user;

const chineseLevels = [
    'Beginner',
    'Advanced',
    'Native/Fluent',
    'HSK 1',
    'HSK 2',
    'HSK 3',
    'HSK 4',
    'HSK 5',
    'HSK 6',
    'TOCFL N1',
    'TOCFL N2',
    'TOCFL 1',
    'TOCFL 2',
    'TOCFL 3',
    'TOCFL 4',
    'TOCFL 5',
    'TOCFL 6',
];
const formattedChineseLevels = chineseLevels.map(level => ({ value: level, label: level }));
const countrySearchTerm = ref(props.user.country ? props.countries.find(c => c.alpha2 === props.user.country)?.name : ''); // Input display value
const showCountrySuggestions = ref(false);

const filteredCountries = computed(() => {
    if (!countrySearchTerm.value || !props.countries) {
        return [];
    }
    const lowerCaseSearchTerm = countrySearchTerm.value.toLowerCase();
    return props.countries.filter(country =>
        country.name.toLowerCase().includes(lowerCaseSearchTerm)
    ).slice(0, 8);
});

const handleCountryInputBlur = () => {
    setTimeout(() => {
        showCountrySuggestions.value = false;
    }, 150);
};

const selectCountry = (country) => {
    countrySearchTerm.value = country.name;
    form.country = country.alpha2;
    showCountrySuggestions.value = false;
};
</script>

<template>
    <section>
        <header>
            <h2 class="text-lg font-medium text-gray-900">Profile Information</h2>
            <p class="mt-1 text-sm text-gray-600">
                Update your account's profile information and email address.
            </p>
        </header>

        <form @submit.prevent="form.patch(route('Profile.update-information'))" class="mt-6 space-y-6">
            <div>
                <label for="username" class="block font-medium text-sm text-gray-700">Username</label>
                <Input id="username" type="text" required autofocus :tabindex="1" autocomplete="username"
                    v-model="form.username" placeholder="Your username" />
                <div v-if="form.errors.username" class="text-red-600 text-sm mt-1">
                    {{ form.errors.username }}
                </div>
            </div>

            <div>
                <label for="first_name" class="block font-medium text-sm text-gray-700">First Name</label>
                <Input id="first_name" type="text" required autofocus :tabindex="2" autocomplete="given-name"
                    v-model="form.first_name" placeholder="Your first name" />
                <div v-if="form.errors.first_name" class="text-red-600 text-sm mt-1">
                    {{ form.errors.first_name }}
                </div>
            </div>

            <div>
                <label for="last_name" class="block font-medium text-sm text-gray-700">Last Name</label>
                <Input id="last_name" type="text" required autofocus :tabindex="3" autocomplete="family-name"
                    v-model="form.last_name" placeholder="Your last name" />
                <div v-if="form.errors.last_name" class="text-red-600 text-sm mt-1">
                    {{ form.errors.last_name }}
                </div>
            </div>

            <div>
                <label for="email" class="block font-medium text-sm text-gray-700">Email</label>
                <Input id="email" type="text" required autofocus :tabindex="3" autocomplete="email" v-model="form.email"
                    placeholder="Your email" />
                <div v-if="form.errors.email" class="text-red-600 text-sm mt-1">
                    {{ form.errors.email }}
                </div>
            </div>

            <div class="relative">
                <label for="country_search" class="block font-medium text-sm text-gray-700">Country</label>
                <Input id="country_search" type="text" required autofocus :tabindex="4" autocomplete="off"
                    @focus="showCountrySuggestions = true" @blur="handleCountryInputBlur" v-model="countrySearchTerm"
                    placeholder="Your country" />
                <InputError class="mt-2" :message="form.errors.country" />

                <ul v-if="showCountrySuggestions && filteredCountries.length"
                    class="text-gray-900 absolute z-10 w-full bg-white border border-gray-300 rounded-md shadow-lg max-h-60 overflow-y-auto mt-1">
                    <li v-for="country in filteredCountries" :key="country.alpha2"
                        @mousedown.prevent="selectCountry(country)" class="px-4 py-2 cursor-pointer hover:bg-gray-100">
                        {{ country.name }} {{ country.emoji }}
                    </li>
                </ul>
            </div>

            <div>
                <label for="city" class="block font-medium text-sm text-gray-700">City</label>
                <Input id="city" type="text" required autofocus :tabindex="5" autocomplete="address-level2"
                    v-model="form.city" placeholder="Your city" />
                <div v-if="form.errors.city" class="text-red-600 text-sm mt-1">
                    {{ form.errors.city }}
                </div>
            </div>

            <div>
                <label for="native_language" class="block font-medium text-sm text-gray-700">Native Language</label>
                <Input id="native_language" type="text" required autofocus :tabindex="5" autocomplete="language"
                    v-model="form.native_language" placeholder="Your language" />
                <div v-if="form.errors.native_language" class="text-red-600 text-sm mt-1">
                    {{ form.errors.native_language }}
                </div>
            </div>

            <div>
                <label for="chinese_level" class="block font-medium text-sm text-gray-700">Chinese Level</label>
                <Select
                id="chinese_level"
                v-model="form.chinese_level"
                :options="formattedChineseLevels"
                placeholder="Select your level"
                class="mt-1 block"
            />
                <div v-if="form.errors.chinese_level" class="text-red-600 text-sm mt-1">
                    {{ form.errors.chinese_level }}
                </div>
            </div>

            <div v-if="authUser.email_verified_at === null">
                <p class="text-sm mt-2 text-gray-800">
                    Your email address is unverified.
                    <Link :href="route('verification.send')" method="post" as="button"
                        class="underline text-sm text-gray-600 hover:text-gray-900 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
                        @success="verificationLinkSent = true">
                    Click here to re-send the verification email.
                    </Link>
                </p>

                <div v-show="verificationLinkSent" class="mt-2 font-medium text-sm text-green-600">
                    A new verification link has been sent to your email address.
                </div>
            </div>

            <div class="flex items-center gap-4">
                <button type="submit" :disabled="form.processing"
                    class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-700 focus:bg-gray-700 active:bg-gray-900 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 transition ease-in-out duration-150">
                    Save
                </button>

                <Transition enter-active-class="transition ease-in-out" enter-from-class="opacity-0"
                    leave-active-class="transition ease-in-out" leave-to-class="opacity-0">
                    <p v-if="form.isDirty" class="text-sm text-gray-600">Changed.</p>
                    <p v-else-if="form.recentlySuccessful" class="text-sm text-gray-600">Saved.</p>
                </Transition>
            </div>
        </form>
    </section>
</template>