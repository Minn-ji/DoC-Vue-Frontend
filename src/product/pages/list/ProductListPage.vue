<template>
    <v-container>
        <v-row class="justify-end mb-4 align-center">
            <v-col cols="3">
                <v-select v-model="selectedCategory" :items="categories" label="카테고리 선택"
                    class="category-select custom-select" style="margin-bottom: 0;">
                </v-select>
            </v-col>
            <v-col cols="auto">
                <v-text-field v-model="searchQuery" label="검색" clearable class="search-input custom-select"
                    style="margin-bottom: 0; max-width: 300px;">
                </v-text-field>
            </v-col>
            <v-col v-if="isSeller" cols="auto">
                <v-btn :to="{ name: 'ProductRegisterPage' }" class="mb-5" height="40">
                    상품 등록
                </v-btn>
            </v-col>
        </v-row>
        <h1 @click="toggleVisibility('recommendationVisible')">추천 이모티콘<span class="small-icon">></span></h1>
        <v-row v-if="recommendationVisible && recommendProductList.length > 0">
            <v-col v-for="(product, index) in recommendProductList" :key="index" cols="12" sm="6" md="4" lg="3">
                <v-card @click="goToProductReadPage(product.productId)">
                    <v-img :src="getImageUrl(product.productTitleImage)" aspect-ratio="1.5">
                        <template v-slot:placeholder>
                            <v-row class="fill-height ma-0" align="center" justify="center">
                                <v-progress-circular indeterminate color="grey lighten-5" />
                            </v-row>
                        </template>
                    </v-img>
                    <v-card-title>{{ product.productName }}</v-card-title>
                    <v-card-subtitle>{{ product.productPrice }}</v-card-subtitle>
                </v-card>
            </v-col>
        </v-row>
        <v-row v-else-if="recommendationVisible">
            <v-col cols="12" class="text-center">
                <v-alert type="info">추천 상품이 없습니다!</v-alert>
            </v-col>
        </v-row>
        <h1 @click="toggleVisibility('allProductsVisible')">전체 이모티콘<span class="small-icon">></span></h1>
        <v-row v-if="allProductsVisible && paginatedProducts.length > 0">
            <v-col v-for="(product, index) in paginatedProducts" :key="index" cols="12" sm="6" md="4" lg="3">
                <v-card @click="goToProductReadPage(product.productId)">
                    <v-img :src="getImageUrl(product.productTitleImage)" aspect-ratio="1.5">
                        <template v-slot:placeholder>
                            <v-row class="fill-height ma-0" align="center" justify="center">
                                <v-progress-circular indeterminate color="grey lighten-5" />
                            </v-row>
                        </template>
                    </v-img>
                    <v-card-title>{{ product.productName }}</v-card-title>
                    <v-card-subtitle>{{ product.productPrice }}</v-card-subtitle>
                </v-card>
            </v-col>
        </v-row>
        <v-row v-else-if="allProductsVisible">
            <v-col cols="12" class="text-center">
                <v-alert type="info">등록된 전체 이모티콘이 없습니다!</v-alert>
            </v-col>
        </v-row>
        <v-row v-if="filteredProducts.length > itemsPerPage">
            <v-col cols="12" class="text-center">
                <v-pagination v-model="currentPage" :length="Math.ceil(filteredProducts.length / itemsPerPage)"
                    @input="changePage" />
            </v-col>
        </v-row>
        <v-spacer class="my-10"></v-spacer>
        <footer style="text-align: center;">
            <h5> 이용약관 | 유료이용안내 | 개인정보처리방침 | 기업고객 | 문의하기 | 공정위사업자정보 | (주)춘식 </h5>
        </footer>
    </v-container>
</template>


<script>
import { mapActions, mapState } from 'vuex'

const productModule = 'productModule'
const accountModule = 'accountModule'

export default {
    computed: {
        ...mapState(productModule, ['products', 'recommendProductList']),
        paginatedProducts() {
            // 현재 페이지의 상품 목록 계산
            const startIndex = (this.currentPage - 1) * this.itemsPerPage;
            return this.filteredProducts.slice(startIndex, startIndex + this.itemsPerPage);
        },
        filteredProducts() {
            let products = this.products;

            if (this.selectedCategory !== '전체') {
                products = products.filter(product => product.productCategory === this.selectedCategory);
            }

            if (this.searchQuery) {
                const query = this.searchQuery.toLowerCase();
                products = products.filter(product => product.productName.toLowerCase().includes(query));
            }

            return products;
        }
    },
    mounted() {
        this.requestProductListToDjango();
        this.defineRecommendProductIdList();
        this.requestRecommendProductListToDjango(this.recommendProductIdList);
        this.checkRoleType();
    },
    methods: {
        ...mapActions(productModule, [
            'requestProductListToDjango',
            'requestRecommendProductListToDjango'
        ]),
        ...mapActions(accountModule, ['requestRoleTypeToDjango']),
        async checkRoleType() {
            try {
                const roleType = await this.requestRoleTypeToDjango();
                console.log('roleType:', roleType);
                this.isSeller = roleType === 'SELLER';
            } catch (error) {
                console.log('사업자가 아닙니다!');
            }
        },
        getImageUrl(imageName) {
            return require(`@/assets/images/uploadImages/${imageName}`);
        },
        goToProductReadPage(productId) {
            this.$router.push({
                name: 'ProductReadPage',
                params: { productId }
            });
        },
        toggleVisibility(section) {
            this[section] = !this[section];
        },
        defineRecommendProductIdList() {
            this.recommendProductIdList = localStorage.getItem('recommendProductIdList');

            if (this.recommendProductIdList) {
                this.recommendProductIdList = this.recommendProductIdList.split(',');
                this.recommendProductIdList = this.recommendProductIdList.map(item => parseInt(item, 10));
                console.log('recommendProductIdList:', this.recommendProductIdList);
            } else {
                console.error('LocalStorage에서 데이터를 찾을 수 없습니다.');
            }
        },
        changePage(page) {
            this.currentPage = page;
        }
    },
    data() {
        return {
            categories: ['전체', '귀여운', '재밌는', '메시지'],
            selectedCategory: '전체',
            searchQuery: '', // 검색어를 저장하는 변수
            isSeller: false, // 판매자 여부를 저장하는 상태 변수
            recommendationVisible: true,
            allProductsVisible: true,
            recommendProductIdList: [],
            currentPage: 1,
            itemsPerPage: 8 // 한 페이지에 보여질 상품 개수
        };
    }
};
</script>

<style>
.search-input {
    width: 300px;
}

.custom-select .v-input__control {
    background-color: #f4f6eb3a;
    color: #a71616;
}

.custom-select .v-select__selections {
    color: #333;
}

.custom-select .v-label {
    color: #a71616;
}

.mb-5 {
    background-color: #deed933a;
}

.small-icon {
    font-size: 40px;
    margin-left: 8px;
    color: #3f4144;
    vertical-align: middle;
    position: relative;
    top: -10px;
    font-weight: 300;
    opacity: 0.6;
}

.title {
    color: #0b0b0b;
    font-family: 'Roboto', sans-serif;
    font-weight: bold;
}

.v-card-subtitle {
    color: #a01f1f;
    font-weight: bold;
}
</style>

<head>
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
</head>
