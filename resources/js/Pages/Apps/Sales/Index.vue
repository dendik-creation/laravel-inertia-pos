<template>
    <Head>
        <title>Laporan Penjualan - Aplikasi Kasir</title>
    </Head>
    <main class="c-main">
        <div class="container-fluid">
            <div class="fade-in">
                <div class="row">
                    <div class="col-md-12">
                        <div
                            class="card border-0 rounded-3 shadow border-top-purple"
                        >
                            <div class="card-header">
                                <span class="font-weight-bold"
                                    ><i class="fa fa-chart-bar"></i> LAPORAN
                                    PENJUALAN</span
                                >
                            </div>
                            <div class="card-body">
                                <form @submit.prevent="filter">
                                    <div class="row">
                                        <div class="col-md-9">
                                            <div class="mb-3">
                                                <label
                                                    class="form-label fw-bold"
                                                    >RENTANG WAKTU</label
                                                >
                                                <input
                                                    type="date"
                                                    id="date-range-filter"
                                                    v-model="date_range"
                                                    class="form-control"
                                                />
                                            </div>
                                        </div>
                                        <div
                                            :class="{
                                                'col-md-2': has_filter,
                                                'col-md-3': !has_filter,
                                            }"
                                        >
                                            <div class="mb-3">
                                                <label
                                                    class="form-label fw-bold text-white"
                                                    >*</label
                                                >
                                                <button
                                                    class="btn btn-md btn-purple border-0 shadow w-100"
                                                >
                                                    <i class="fa fa-filter"></i>
                                                    FILTER
                                                </button>
                                            </div>
                                        </div>
                                        <div class="col-md-1" v-if="has_filter">
                                            <div class="mb-3">
                                                <label
                                                    class="form-label fw-bold text-white"
                                                    >*</label
                                                >
                                                <button
                                                    type="button"
                                                    @click="clearFilter"
                                                    class="btn btn-md btn-danger border-0 shadow w-100"
                                                >
                                                    <i
                                                        class="fa-solid fa-circle-xmark"
                                                    ></i>
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </form>

                                <div v-if="sales">
                                    <hr />
                                    <div class="export text-end mb-3">
                                        <a
                                            :href="`/apps/sales/export?start_date=${start_date}&end_date=${end_date}`"
                                            target="_blank"
                                            class="btn btn-success btn-md border-0 shadow me-3"
                                            ><i class="fa fa-file-excel"></i>
                                            EXCEL</a
                                        >
                                        <a
                                            :href="`/apps/sales/pdf?start_date=${start_date}&end_date=${end_date}`"
                                            target="_blank"
                                            class="btn btn-secondary btn-md border-0 shadow"
                                            ><i class="fa fa-file-pdf"></i>
                                            PDF</a
                                        >
                                    </div>
                                    <table class="table table-bordered">
                                        <thead>
                                            <tr
                                                style="
                                                    background-color: #e6e6e7;
                                                "
                                            >
                                                <th scope="col">Tanggal</th>
                                                <th scope="col">Invoice</th>
                                                <th scope="col">Kasir</th>
                                                <th scope="col">Customer</th>
                                                <th scope="col">Total</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr
                                                v-for="sale in sales"
                                                :key="sale.id"
                                            >
                                                <td>{{ sale.created_at }}</td>
                                                <td class="text-center">
                                                    {{ sale.invoice }}
                                                </td>
                                                <td>{{ sale.cashier.name }}</td>
                                                <td>
                                                    {{
                                                        sale.customer
                                                            ? sale.customer.name
                                                            : "Umum"
                                                    }}
                                                </td>
                                                <td class="text-end">
                                                    Rp.
                                                    {{
                                                        formatPrice(
                                                            sale.grand_total
                                                        )
                                                    }}
                                                </td>
                                            </tr>
                                            <tr>
                                                <td
                                                    colspan="4"
                                                    class="text-end fw-bold"
                                                    style="
                                                        background-color: #e6e6e7;
                                                    "
                                                >
                                                    TOTAL
                                                </td>
                                                <td
                                                    class="text-end fw-bold"
                                                    style="
                                                        background-color: #e6e6e7;
                                                    "
                                                >
                                                    Rp. {{ formatPrice(total) }}
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>
</template>

<script>
import LayoutApp from "../../../Layouts/App.vue";
import { Head, Link } from "@inertiajs/inertia-vue3";
import { onMounted, ref, watch } from "vue";
import { Inertia } from "@inertiajs/inertia";
// Flatpickr
import flatpickr from "flatpickr";
import "flatpickr/dist/flatpickr.css";
import "flatpickr/dist/themes/airbnb.css";

export default {
    //layout App
    layout: LayoutApp,

    //register components
    components: {
        Head,
        Link,
    },

    //props
    props: {
        errors: Object,
        sales: Array,
        total: Number,
    },

    setup() {
        const start_date = ref("");
        const end_date = ref("");
        const date_range = ref(null);
        const has_filter = ref(false);

        onMounted(() => {
            flatpickr("#date-range-filter", {
                mode: "range",
                dateFormat: "Y-m-d",
                altFormat: "d F Y",
                altInput: true,
                onChange: (date, string_date) => {
                    separateDate(string_date);
                },
                defaultDate: generateDefaultDate(),
                locale: {
                    rangeSeparator: "  >>  ",
                },
            });

            start_date.value = generateDefaultDate()[0];
            end_date.value = generateDefaultDate()[1];
        });

        watch(start_date, () => {
            if (
                new URL(document.location).searchParams.get("start_date") &&
                new URL(document.location).searchParams.get("end_date")
            ) {
                has_filter.value = true;
            }
        });
        const separateDate = (dateString) => {
            let dates = dateString.split("  >>  ");
            start_date.value = dates[0];
            end_date.value = dates[1];
        };

        const generateDefaultDate = () => {
            let year = new Date().getFullYear();
            let month = new Date().getMonth();

            let firstDay =
                new URL(document.location).searchParams.get("start_date") ||
                new Date(year, month, 1).toLocaleDateString("en-CA");
            let lastDay =
                new URL(document.location).searchParams.get("end_date") ||
                new Date(year, month + 1, 0).toLocaleDateString("en-CA");
            return [firstDay, lastDay];
        };

        const clearFilter = () => {
            Inertia.get("/apps/sales");
        };

        const filter = () => {
            Inertia.get("/apps/sales/filter", {
                start_date: start_date.value,
                end_date: end_date.value,
            });
        };

        return {
            start_date,
            end_date,
            date_range,
            has_filter,
            generateDefaultDate,
            separateDate,
            clearFilter,
            filter,
        };
    },
};
</script>

<style></style>
