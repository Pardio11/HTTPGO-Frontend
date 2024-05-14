<script setup>
import { ref, onMounted } from 'vue';
const cars = ref([]);
const title = ref("Add New Car")
const keyCar = ref("")
const isOpen = ref(false);

const newCar = ref({
    brand: '',
    model: '',
    year: null,
    motor: {
        size: null,
        horsepower: null,
        torque: null,
        max_rpm: null
    }
});
const url = "https://go-http-redis-production.up.railway.app/"
onMounted(async () => {
    loadCars()
});

const openModal = () => {
    title.value = "Add New Car"
    isOpen.value = true;
};

const closeModal = () => {
    isOpen.value = false;
};
const edit = async (key) => {
    title.value = "Update Car: " + key
    keyCar.value = key
    try {
        const response = await fetch(url + 'cars/' + key);
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        newCar.value = data;
        console.log(newCar.value)
    } catch (error) {
        console.error('Error fetching data:', error);
    }
    isOpen.value = true;
};

const submit = () => {
    if (title.value == "Add New Car") {
        addCar()
    } else {

        updateCar(keyCar.value)
    }

};
const loadCars = async () => {
    try {
        const response = await fetch(url + 'cars');
        if (!response.ok) {
            throw new Error('Network response was not ok');
        }
        const data = await response.json();
        cars.value = data;
        console.log(cars.value)
    } catch (error) {
        console.error('Error fetching data:', error);
    }
}
const addCar = async () => {
    try {
        title.value = "Add New Car"
        const jsonString = JSON.stringify(newCar._rawValue);
        const response = await fetch(url + 'cars', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: jsonString
        });

        if (!response.ok) {
            throw new Error('Failed to add car');
        }
        const addedCar = await response.json();
        closeModal();
        loadCars()
    } catch (error) {
        console.error('Error adding car:', error.message);
    }
};

const deleteCar = async (carId) => {
    try {
        const response = await fetch(url + `cars/${carId}`, {
            method: 'DELETE',
            headers: {
                'Content-Type': 'application/json'
            },
        });
        delete cars.value[carId];
        if (!response.ok) {
            throw new Error('Failed to delete car');
        }

    } catch (error) {
        console.error('Error deleting car:', error);
    }
};


const updateCar = async (carId) => {

    try {

        const jsonString = JSON.stringify(newCar._rawValue);
        const response = await fetch(url + `cars/${carId}`, {
            method: 'PUT',
            headers: {
                'Content-Type': 'application/json'
            },
            body: jsonString
        });

        if (!response.ok) {
            throw new Error('Failed to add car');
        }
        const addedCar = await response.json();
        loadCars()
        closeModal();
    } catch (error) {
        console.error('Error adding car:', error.message);
    }
};

</script>

<template>
    <div class="w-[80%] bg-slate-900 rounded-md">
        <div class="w-full flex justify-between items-center py-3">
            <h1 class="text-3xl text-stone-200 font-medium text-center flex-grow">Car CRUD (GO & Redis)</h1>
            <button @click="openModal" class="bg-stone-700 rounded-xl py-1 px-2 text-white mr-8 transition duration-300 ease-in-out 
                transform hover:bg-stone-800 focus:outline-none focus:ring-2 focus:ring-stone-500">
                Add Car
            </button>
        </div>
        <table class=" border-t border-white w-full text-white text-lg ">
            <thead class=" bg-black min-h-28 ">
                <tr>
                    <th scope="col" class="font-normal py-4 pl-3">Key</th>
                    <th scope="col" class="font-normal py-4">Brand</th>
                    <th scope="col" class="font-normal py-4">Model</th>
                    <th scope="col" class="font-normal py-4">Year</th>
                    <th scope="col" class="font-normal py-4">Motor Size</th>
                    <th scope="col" class="font-normal py-4">Horse Power</th>
                    <th scope="col" class="font-normal py-4">Torque</th>
                    <th scope="col" class="font-normal py-4">Max RPM</th>
                    <th scope="col" class="font-normal py-4 pr-3"></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(car, key) in cars" :key="key" class="border-b border-gray-700">
                    <td class="py-3 text-center">
                        {{ key }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.brand }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.model }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.year }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.motor.size }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.motor.horsepower }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.motor.torque }}
                    </td>
                    <td class="py-3 text-center">
                        {{ car.motor.max_rpm }}
                    </td>
                    <td class="py-3 text-center ">
                        <button @click="edit(key)"
                            class="font-bold rounded-full transition duration-300 ease-in-out transform hover:scale-105 mr-4">
                            <NuxtImg src="./imgs/edit.png" alt="Delete" class="h-6 w-6 inline-block" />
                        </button>
                        <button @click="deleteCar(key)"
                            class="font-bold rounded-full transition duration-300 ease-in-out transform hover:scale-105 ">
                            <NuxtImg src="./imgs/trash.svg" alt="Delete" class="h-6 w-6 inline-block" />
                        </button>
                    </td>
                </tr>
            </tbody>

        </table>
    </div>

    <div v-if="isOpen" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50">

        <div class="bg-white p-8 rounded shadow-lg ">
            <h2 class="text-lg font-semibold mb-4 w-full">{{ title }}</h2>
            <form @submit.prevent="submit()">
                <div class=" grid grid-cols-2 gap-4">
                    <div>
                        <div class="mb-4">
                            <label for="brand" class="block mb-1">Brand:</label>
                            <input type="text" id="brand" v-model="newCar.brand"
                                class="w-full border rounded px-3 py-2">
                        </div>
                        <div class="mb-4 " v-if="title == 'Add New Car'">
                            <label for="model" class="block mb-1">Model:</label>
                            <input type="text" id="model" v-model="newCar.model"
                                class="w-full border rounded px-3 py-2">
                        </div>
                        <div class="mb-4" v-if="title == 'Add New Car'">
                            <label for="year" class="block mb-1">Year:</label>
                            <input type="number" id="year" v-model="newCar.year"
                                class="w-full border rounded px-3 py-2">
                        </div>
                        <div class="mb-4">
                            <label for="motorSize" class="block mb-1">Motor Size:</label>
                            <input type="number" step=".01" id="motorSize" v-model="newCar.motor.size"
                                class="w-full border rounded px-3 py-2">
                        </div>

                    </div>
                    <div class="h-full flex flex-col">
                        <div>
                            <div class="mb-4">
                                <label for="horsepower" class="block mb-1">Horsepower:</label>
                                <input type="number" id="horsepower" v-model="newCar.motor.horsepower"
                                    class="w-full border rounded px-3 py-2">
                            </div>
                            <div class="mb-4">
                                <label for="torque" class="block mb-1">Torque:</label>
                                <input type="number" id="torque" v-model="newCar.motor.torque"
                                    class="w-full border rounded px-3 py-2">
                            </div>
                            <div class="mb-4">
                                <label for="maxRpm" class="block mb-1">Max RPM:</label>
                                <input type="number" id="maxRpm" v-model="newCar.motor.max_rpm"
                                    class="w-full border rounded px-3 py-2">
                            </div>
                        </div>
                        <div class="mt-auto mb-4 flex">
                            <button type="submit" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded">Add
                                Car</button>
                            <button @click="closeModal" type="button"
                                class="bg-gray-200 text-gray-700 px-4 py-2 rounded ml-2">Cancel</button>
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </div>

</template>
