<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { Link } from '@inertiajs/vue3';

defineProps({
    books: Object,
});
</script>

<template>
    <AuthenticatedLayout>
        <h1>Data Buku</h1>

        <!-- Link menuju form tambah buku, akan diarahkan ke Books/Create.vue (Langkah 7) -->
        <Link href="/books/create">+ Tambah Buku</Link>

        <table>
            <thead>
                <tr>
                    <th>Kode</th>
                    <th>Judul</th>
                    <th>Pengarang</th>
                    <th>Stok</th>
                    <th>Tahun</th>
                </tr>
            </thead>
            <tbody>
                <!-- v-for mengulang setiap data buku, sama seperti perulangan foreach -->
                <tr v-for="book in books.data" :key="book.id">
                    <td>{{ book.kode_buku }}</td>
                    <td>{{ book.judul }}</td>
                    <td>{{ book.pengarang }}</td>
                    <td>{{ book.stok }}</td>
                    <td>{{ book.tahun }}</td>
                </tr>
            </tbody>
        </table>
    </AuthenticatedLayout>
</template>