<script setup lang="ts">
import { Link, usePage } from '@inertiajs/vue3';
import { computed, ref } from 'vue';

const page = usePage();
const user = computed(() => page.props.auth.user);
const isAdmin = computed(() => user.value && user.value.is_admin);

const showingNavigationDropdown = ref(false);
const toggleNavDropdown = () => {
    showingNavigationDropdown.value = !showingNavigationDropdown.value;
};

const navLinks = [
    { name: 'Dashboard', route: 'dashboard', requiresAuth: true },
    // { name: 'My Words', route: 'words.index', requiresAuth: true },
    { name: 'Profile', route: 'Profile.edit', requiresAuth: true },
    { name: 'Admin', route: 'admin.dashboard', requiresAuth: true, requiresAdmin: true },
    // { name: 'Log In', route: 'login', requiresAuth: false, isPrimary: true },
    { name: 'Register', route: 'register', requiresAuth: false, isBordered: true },
    { name: 'Log Out', route: 'logout', requiresAuth: true, isLogout: true },
];
</script>

<template>
    <nav class="bg-gray-800 text-white p-4 shadow-md">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between h-16 items-center">
                <!-- Left side: App Name / Logo -->
                <div class="flex-shrink-0 flex items-center">
                    <Link :href="user ? (isAdmin ? route('admin.dashboard') : route('dashboard')) : route('login')"
                        class="text-2xl font-bold text-white hover:text-gray-200 transition">
                        <div class="flex items-center">
                            <img src="/logo_HanyuVocabTrackerWhite.png" alt="Hanyu VocabTracker Logo"
                                class="h-16 w-auto mr-3">
                        </div>
                    </Link>
                </div>

                <!-- Desktop Navigation Links -->
                <div class="hidden md:flex items-center space-x-4">
                    <template v-for="link in navLinks" :key="link.name">
                        <!-- Add condition for requiresAdmin -->
                        <template v-if="link.requiresAuth === !!user && (!link.requiresAdmin || isAdmin)">
                            <!-- Regular Links -->
                            <Link v-if="!link.isPrimary && !link.isBordered && !link.isLogout" :href="route(link.route)"
                                class="px-3 py-2 hover:bg-gray-700 rounded-md transition">
                            {{ link.name }}
                            </Link>

                            <!-- Primary/Login Button Style -->
                            <Link v-else-if="link.isPrimary" :href="route(link.route)"
                                class="px-3 py-2 bg-indigo-600 hover:bg-indigo-700 rounded-md font-semibold transition">
                            {{ link.name }}
                            </Link>

                            <!-- Bordered/Register Button Style -->
                            <Link v-else-if="link.isBordered" :href="route(link.route)"
                                class="px-3 py-2 border border-white hover:border-gray-300 hover:bg-gray-700 rounded-md transition">
                            {{ link.name }}
                            </Link>

                            <!-- Logout Button -->
                            <Link v-else-if="link.isLogout" :href="route(link.route)" method="post" as="button"
                                class="px-3 py-2 bg-red-600 hover:bg-red-700 rounded-md font-semibold transition">
                            {{ link.name }}
                            </Link>
                        </template>
                    </template>
                </div>

                <!-- Mobile Hamburger Button -->
                <div class="-mr-2 flex items-center md:hidden">
                    <button @click="toggleNavDropdown"
                        class="inline-flex items-center justify-center p-2 rounded-md text-gray-400 hover:text-white hover:bg-gray-700 focus:outline-none focus:bg-gray-700 focus:text-white transition duration-150 ease-in-out"
                        aria-label="Toggle navigation">
                        <svg class="h-6 w-6" :class="{
                            'hidden': showingNavigationDropdown,
                            'inline-flex': !showingNavigationDropdown,
                        }" stroke="currentColor" fill="none" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M4 6h16M4 12h16M4 18h16" />
                        </svg>
                        <svg class="h-6 w-6" :class="{
                            'hidden': !showingNavigationDropdown,
                            'inline-flex': showingNavigationDropdown,
                        }" stroke="currentColor" fill="none" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>
            </div>
        </div>

        <!-- Mobile Navigation Menu -->
        <Transition enter-active-class="transition ease-out duration-200"
            enter-from-class="transform opacity-0 scale-95" enter-to-class="transform opacity-100 scale-100"
            leave-active-class="transition ease-in duration-75" leave-from-class="transform opacity-100 scale-100"
            leave-to-class="transform opacity-0 scale-95">
            <div v-show="showingNavigationDropdown" class="md:hidden">
                <div class="pt-2 pb-3 space-y-1">
                    <template v-for="link in navLinks" :key="link.name + '-mobile'">
                        <!-- Add condition for requiresAdmin -->
                        <template v-if="link.requiresAuth === !!user && (!link.requiresAdmin || isAdmin)">
                            <!-- Regular Links -->
                            <Link v-if="!link.isPrimary && !link.isBordered && !link.isLogout" :href="route(link.route)"
                                class="block w-full pl-3 pr-4 py-2 border-l-4 border-transparent text-left text-base font-medium text-gray-300 hover:text-white hover:bg-gray-700 hover:border-gray-500 focus:outline-none focus:text-white focus:bg-gray-700 focus:border-gray-500 transition duration-150 ease-in-out"
                                @click="showingNavigationDropdown = false">
                            {{ link.name }}
                            </Link>

                            <!-- Primary/Login Button Style -->
                            <Link v-else-if="link.isPrimary" :href="route(link.route)"
                                class="block w-full text-center px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded-md font-semibold transition mt-2 mx-auto"
                                @click="showingNavigationDropdown = false">
                            {{ link.name }}
                            </Link>

                            <!-- Bordered/Register Button Style -->
                            <Link v-else-if="link.isBordered" :href="route(link.route)"
                                class="block w-full text-center px-4 py-2 border border-white hover:border-gray-300 hover:bg-gray-700 rounded-md transition mt-2 mx-auto"
                                @click="showingNavigationDropdown = false">
                            {{ link.name }}
                            </Link>

                            <!-- Logout Button -->
                            <Link v-else-if="link.isLogout" :href="route(link.route)" method="post" as="button"
                                class="block w-full text-center px-4 py-2 bg-red-600 hover:bg-red-700 rounded-md font-semibold transition mt-2 mx-auto"
                                @click="showingNavigationDropdown = false">
                            {{ link.name }}
                            </Link>
                        </template>
                    </template>
                </div>
            </div>
        </Transition>
    </nav>
</template>