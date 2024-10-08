<template>
    <div>
        <p>{{ displayHours }}:{{ displayMinutes }}:{{ displaySeconds }}</p>
        <p v-if="noDataMessage">{{ noDataMessage }}</p>
    </div>
</template>

<script>
import axios from 'axios';
import Swal from 'sweetalert2';

export default {
    data() {
        return {
            displayHours: "00",
            displayMinutes: "00",
            displaySeconds: "00",
            timer: null,
            end: null,
            noDataMessage: "", // Variable to store message if no data is received
            dataCheckInterval: null,
        };
    },
    computed: {
        _seconds() {
            return 1000;
        },
        _minutes() {
            return this._seconds * 60;
        },
        _hours() {
            return this._minutes * 60;
        }
    },
    mounted() {
        this.loadEndTime();
        this.checkForData();
        this.dataCheckInterval = setInterval(this.checkForData, 10 * this._minutes); // Check for new data every 10 minutes
    },
    beforeDestroy() {
        clearInterval(this.dataCheckInterval); // Clear interval when component is destroyed
        if (this.timer) {
            clearInterval(this.timer);
        }
    },
    methods: {
        formatNum(num) {
            return num < 10 ? "0" + num : num;
        },
        showRemaining() {
            this.timer = setInterval(() => {
                const now = new Date();
                const distance = this.end.getTime() - now.getTime();

                if (distance < 0) {
                    clearInterval(this.timer);
                    this.timer = null;
                    this.displayHours = "00";
                    this.displayMinutes = "00";
                    this.displaySeconds = "00";
                    localStorage.removeItem('end');
                    this.notifyTimerFinished();
                    return;
                }
                const hours = Math.floor(distance / this._hours);
                const minutes = Math.floor((distance % this._hours) / this._minutes);
                const seconds = Math.floor((distance % this._minutes) / this._seconds);
                this.displayHours = this.formatNum(hours);
                this.displayMinutes = this.formatNum(minutes);
                this.displaySeconds = this.formatNum(seconds);
            }, 1000);
        },
        startTimer() {
            const now = new Date();
            this.end = new Date(now.getTime() + 240  * this._minutes); // Set timer for 240 minutes (4 hours) from now
            localStorage.setItem('end', this.end.toISOString()); // Store the end time in localStorage in ISO format
            if (!this.timer) {
                this.showRemaining();
            }
        },
        loadEndTime() {
            const endTime = localStorage.getItem('end');
            if (endTime) {
                this.end = new Date(endTime);
                if (this.end.getTime() > new Date().getTime()) {
                    this.showRemaining();
                } else {
                    localStorage.removeItem('end');
                }
            }
        },
        stopTimer() {
            if (this.timer) {
                clearInterval(this.timer);
                this.timer = null;
                this.displayHours = "00";
                this.displayMinutes = "00";
                this.displaySeconds = "00";
                localStorage.removeItem('end');
            }
        },
        notifyTimerFinished() {
            Swal.fire({
                title: "Berhasil",
                text: "Waktu pengering gabah telah berhasil dilakukan",
                icon: "success"
            });
        },
        async checkForData() {
            try {
                const response = await axios.get('http://localhost:3000/api/sensor-data'); // Adjust to your API URL

                if (response.data.length > 0) { // Check if there is data from sensors
                    const latestData = response.data[response.data.length - 1]; // Get the latest data
                    const latestTimestamp = new Date(latestData.timestamp); // Assume data has a timestamp property
                    const currentTime = new Date();
                    const timeDiff = (currentTime - latestTimestamp) / 1000; // Time difference in seconds

                    if (timeDiff <= 660) { // Check if the latest data is within the last minute
                        if (!this.timer && !localStorage.getItem('end')) {
                            this.startTimer(); // Start the timer if it is not already running and no end time is stored
                        }
                    } else {
                        this.noDataMessage = 'No recent data received from sensors';
                        this.stopTimer(); // Stop the timer if no recent data is received
                    }
                } else {
                    this.noDataMessage = 'No data received from sensors';
                    this.stopTimer(); // Stop the timer if no data is received
                }
            } catch (error) {
                console.error('Error fetching sensor data:', error);
                this.noDataMessage = 'Error fetching sensor data';
                this.stopTimer(); // Stop the timer if there's an error fetching data
            }
        }
    }
};
</script>
