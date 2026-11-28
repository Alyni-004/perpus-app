<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import { useForm } from '@inertiajs/vue3';
import { ref, watch, onBeforeUnmount } from 'vue';
import axios from 'axios';

const form = useForm({
    kode_buku: '',
    judul: '',
    pengarang: '',
    penerbit: '',
    tahun: '',
    stok: 0,
});

// Validasi realtime kode_buku (Latihan Mandiri Pertemuan 10)
const statusKode = ref('');
let timerKode = null;
let permintaanKode = null;

async function periksaKode() {
    if (permintaanKode) {
        permintaanKode.abort();
    }
    permintaanKode = new AbortController();
    statusKode.value = 'memeriksa';

    try {
        const respons = await axios.get(route('books.cek-kode'), {
            params: {
                kode_buku: form.kode_buku.trim(),
            },
            signal: permintaanKode.signal,
        });
        statusKode.value = respons.data.tersedia ? 'tersedia' : 'terpakai';
    } catch (error) {
        if (axios.isCancel(error)) {
            return;
        }
        statusKode.value = 'gagal';
    }
}

watch(() => form.kode_buku, (nilai) => {
    clearTimeout(timerKode);
    if (!nilai || nilai.trim() === '') {
        if (permintaanKode) {
            permintaanKode.abort();
        }
        statusKode.value = '';
        return;
    }
    timerKode = setTimeout(periksaKode, 500);
});

onBeforeUnmount(() => {
    clearTimeout(timerKode);
    if (permintaanKode) {
        permintaanKode.abort();
    }
});

function submit() {
    form.post(route('books.store'));
}
</script>

<template>
    <AuthenticatedLayout>
        <h1>Tambah Buku</h1>

        <form @submit.prevent="submit">
            <!-- @submit.prevent mencegah form reload halaman, lalu menjalankan function submit() -->

            <label>Kode Buku</label>
            <input v-model="form.kode_buku" type="text">
            <!-- Menampilkan pesan error dari validasi Laravel jika ada -->
            <div v-if="statusKode === 'memeriksa'" class="text-gray-500">
                Memeriksa kode buku...
            </div>
            <div v-else-if="statusKode === 'tersedia'" class="text-green-700">
                Kode buku ini tersedia.
            </div>
            <div v-else-if="statusKode === 'terpakai'" class="text-red-600">
                Kode buku ini sudah terdaftar. Gunakan kode lain.
            </div>
            <div v-else-if="statusKode === 'gagal'" class="text-yellow-700">
                Pemeriksaan otomatis gagal. Anda tetap dapat menyimpan; server akan memeriksa ulang.
            </div>
            <div v-if="form.errors.kode_buku" class="text-red-600">
                {{ form.errors.kode_buku }}
            </div>

            <label>Judul</label>
            <input v-model="form.judul" type="text">
            <span v-if="form.errors.judul">{{ form.errors.judul }}</span>

            <label>Pengarang</label>
            <input v-model="form.pengarang" type="text">
            <span v-if="form.errors.pengarang">{{ form.errors.pengarang }}</span>

            <label>Penerbit</label>
            <input v-model="form.penerbit" type="text">

            <label>Tahun</label>
            <input v-model="form.tahun" type="text">

            <label>Stok</label>
            <input v-model="form.stok" type="number">

            <button type="submit" :disabled="form.processing || statusKode === 'terpakai'">
                Simpan
            </button>
        </form>
    </AuthenticatedLayout>
</template>