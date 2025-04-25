<template>
    <div class="tiles-container">
        <!-- Total Assigned Card -->
        <v-card
            elevation="4"
            color="white"
            variant="elevated"
            class="mx-auto my-4"
            style="width: 330px; height: 175px; cursor: pointer;"
            :class="{ 'selected-card': selectedCard === 'totalAssigned' }"
            @click="selectedCard = 'totalAssigned'"
        >
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Total Assigned</div>
                    <div class="text-caption">Currently assigned seats</div>
                    <p class="text-h4">{{ totalSeats.length }}</p>
                </div>
            </v-card-item>
        </v-card>

        <!-- Assigned But Never Used Card -->
        <v-card
            elevation="4"
            color="white"
            variant="elevated"
            class="mx-auto my-3"
            style="width: 300px; height: 175px; cursor: pointer;"
            :class="{ 'selected-card': selectedCard === 'noshowSeats' }"
            @click="selectedCard = 'noshowSeats'"
        >
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">Assigned But Never Used</div>
                    <div class="text-caption">No show seats</div>
                    <p class="text-h4">{{ noshowSeats }}</p>
                </div>
            </v-card-item>
        </v-card>

        <!-- No Activity in the Last 7 Days Card -->
        <v-card
            elevation="4"
            color="white"
            variant="elevated"
            class="mx-auto my-4"
            style="width: 330px; height: 175px; cursor: pointer;"
            :class="{ 'selected-card': selectedCard === 'unusedIn7Days' }"
            @click="selectedCard = 'unusedIn7Days'"
        >
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">No Activity in the Last 7 days</div>
                    <div class="text-caption">No use in the last 7 days</div>
                    <p class="text-h4">{{ unusedSeatsInSevenDays }}</p>
                </div>
            </v-card-item>
        </v-card>

        <!-- No Activity in the Last 30 Days Card -->
        <v-card
            elevation="4"
            color="white"
            variant="elevated"
            class="mx-auto my-4"
            style="width: 330px; height: 175px; cursor: pointer;"
            :class="{ 'selected-card': selectedCard === 'unusedIn30Days' }"
            @click="selectedCard = 'unusedIn30Days'"
        >
            <v-card-item class="d-flex justify-center align-center">
                <div class="tiles-text">
                    <div class="text-overline mb-1" style="visibility: hidden;">filler</div>
                    <div class="text-h6 mb-1">No Activity in the Last 30 days</div>
                    <div class="text-caption">No use in the last 30 days</div>
                    <p class="text-h4">{{ unusedSeatsInThirtyDays }}</p>
                </div>
            </v-card-item>
        </v-card>
    </div>
    
    <div>
        <v-main class="p-1" style="min-height: 300px;">
            <v-container style="min-height: 300px;" class="px-4 elevation-2">
                <br>
                <h2>All assigned seats </h2>

                <!-- Search Input -->
                <v-text-field
                    v-model="searchQuery"
                    label="Search by Login"
                    clearable
                    class="mb-4"
                ></v-text-field>

                <v-btn color="primary" class="mb-4" @click="downloadCSV">
                    Download CSV
                </v-btn>
                <br>

                <!-- Data Table -->
                <v-data-table
                    :headers="headers"
                    :items="filteredSeats"
                    :items-per-page="10"
                    class="elevation-2"
                >
                    <template #item="{ item, index }">
                        <tr>
                            <td>{{ index + 1 }}</td>
                            <td>{{ item.login }}</td>
                            <td>{{ item.email }}</td> <!-- New Email Column -->
                            <td>{{ item.id }}</td>
                            <td>{{ item.team }}</td>
                            <td>{{ item.created_at }}</td>
                            <td>{{ item.last_activity_at }}</td>
                            <td>{{ item.last_activity_editor }}</td>
                        </tr>
                    </template>
                </v-data-table>
            </v-container>
        </v-main>
    </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watchEffect } from 'vue';
import type { Seat } from '@/model/Seat';

export default defineComponent({
    name: 'SeatsAnalysisViewer',
    props: {
        seats: {
            type: Array as () => Seat[],
            required: true,
            default: () => [],
        },
    },
    setup(props) {
        const totalSeats = ref<Seat[]>([]);
        const searchQuery = ref<string>(''); // Search query state
        const selectedCard = ref<string>('totalAssigned'); // Track the selected card

        // Initialize values for the 3 cards
        const noshowSeats = ref<number>(0); // Assigned But Never Used
        const unusedSeatsInSevenDays = ref<number>(0); // No Activity in the Last 7 days
        const unusedSeatsInThirtyDays = ref<number>(0); // No Activity in the Last 30 days

        // Compute filtered seats based on the selected card
        const filteredSeats = computed(() => {
            let filtered = [];
            if (selectedCard.value === 'totalAssigned') {
                filtered = totalSeats.value;
            } else if (selectedCard.value === 'noshowSeats') {
                filtered = totalSeats.value.filter((seat) => !seat.last_activity_at);
            } else if (selectedCard.value === 'unusedIn7Days') {
                const oneWeekAgo = new Date();
                oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
                filtered = totalSeats.value.filter(
                    (seat) => seat.last_activity_at && new Date(seat.last_activity_at) < oneWeekAgo
                );
            } else if (selectedCard.value === 'unusedIn30Days') {
                const thirtyDaysAgo = new Date();
                thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
                filtered = totalSeats.value.filter(
                    (seat) => seat.last_activity_at && new Date(seat.last_activity_at) < thirtyDaysAgo
                );
            }

            return filtered.filter((seat) =>
                seat.login.toLowerCase().includes(searchQuery.value.toLowerCase())
            );
        });

        // Watch for changes in the seats prop and calculate card values
        watchEffect(() => {
            if (props.seats && Array.isArray(props.seats)) {
                totalSeats.value = props.seats;

                // Calculate values for the 3 cards
                noshowSeats.value = totalSeats.value.filter((seat) => !seat.last_activity_at).length;

                const oneWeekAgo = new Date();
                oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
                unusedSeatsInSevenDays.value = totalSeats.value.filter(
                    (seat) => seat.last_activity_at && new Date(seat.last_activity_at) < oneWeekAgo
                ).length;

                const thirtyDaysAgo = new Date();
                thirtyDaysAgo.setDate(thirtyDaysAgo.getDate() - 30);
                unusedSeatsInThirtyDays.value = totalSeats.value.filter(
                    (seat) => seat.last_activity_at && new Date(seat.last_activity_at) < thirtyDaysAgo
                ).length;
            } else {
                throw new Error('Invalid number of seats');
            }
        });

        return {
            totalSeats,
            searchQuery,
            selectedCard,
            filteredSeats,
            noshowSeats,
            unusedSeatsInSevenDays,
            unusedSeatsInThirtyDays,
        };
    },
    data() {
        return {
            headers: [
                { text: 'S.No', value: 'serialNumber' },
                { text: 'Login', value: 'login' },
                { text: 'Email', value: 'email' }, // New Email Header
                { text: 'GitHub ID', value: 'id' },
                { text: 'Assigning Team', value: 'team' },
                { text: 'Assigned Time', value: 'created_at' },
                { text: 'Last Activity At', value: 'last_activity_at' },
                { text: 'Last Activity Editor', value: 'last_activity_editor' },
            ],
        };
    },
});
</script>

<style scoped>
.selected-card {
    border: 2px solid #1976d2;
}
</style>
