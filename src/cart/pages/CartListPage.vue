<template>
    <v-container>
        <v-row>
            <v-col cols="12">
                <v-card>
                    <v-card-title>Shopping Cart</v-card-title>
                    <v-card-text>
                        <v-table>
                            <thead>
                            <tr>
                                <th>Select</th>
                                <th>Product</th>
                                <th>Price</th>
                                <th>Quantity</th>
                                <th>Total</th>
                                <th>Actions</th>
                            </tr>
                            </thead>
                            <tbody>
                            <tr v-for="item in cartItems" :key="item.cartItemId">
                                <td>
                                    <v-checkbox v-model="selectedItems" :value="item"></v-checkbox>
                                </td>
                                <td>{{ item.productName }}</td>
                                <td>{{ item.productPrice }}</td>
                                <td>
                                    <v-text-field
                                        v-model="item.quantity"
                                        type="number"
                                        min="1"
                                        @change="updateQuantity(item)">
                                    </v-text-field>
                                </td>
                                <td>{{ item.productPrice * item.quantity }}</td>
                                <td>
                                    <v-btn color="red" @click="removeItem(item)">Remove</v-btn>
                                </td>
                            </tr>
                            </tbody>
                        </v-table>
                        <v-divider></v-divider> 
                        <v-row>
                            <v-col>
                                <v-btn color="blue" @click="confirmCheckout">Checkout</v-btn>
                            </v-col>
                            <v-col class="text-right">
                                <strong>Total: {{ selectedItemsTotal }}</strong>
                            </v-col>
                        </v-row>
                    </v-card-text>
                </v-card>
            </v-col>
        </v-row>
        <v-dialog v-model="isCheckoutDialogVisible" max-width="500">
            <v-card>
                <v-card-title>Confirm Checkout</v-card-title>
                <v-card-text>
                    Are you sure you want to order the selected items?
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text 
                        @click="isCheckoutDialogVisible = false">Cancel</v-btn>
                    <v-btn color="blue darken-1" text @click="proceedToOrder">Confirm</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-container>
</template>

<script>
import { mapActions } from "vuex";

const cartModule = 'cartModule'
const orderModule = 'orderModule'

export default {
    data() {
        return {
            cartItems: [],
            selectedItems: [],
            isCheckoutDialogVisible: false,
        };
    },
    computed: {
        cartTotal() {
            if (!Array.isArray(this.cartItems) || this.cartItems.length === 0) {
                return 0;
            }
            return this.cartItems.reduce(
                (total, item) => total + item.productPrice * item.quantity,
                0
            );
        },
        selectedItemsTotal() {
            if (!Array.isArray(this.selectedItems) || this.selectedItems.length === 0) {
                return 0;
            }
            return this.selectedItems.reduce(
                (total, item) => total + item.productPrice * item.quantity,
                0
            );
        },
    },
    methods: {
        ...mapActions(cartModule, ["requestCartListToDjango"]),
        ...mapActions("orderModule", ["requestCreateOrderToDjango"]),
        // updateQuantity(item) {
        // },
        removeItem(item) {
            this.cartItems = this.cartItems.filter(
                cartItem => cartItem.cartItemId !== item.cartItemId);
            this.selectedItems = this.selectedItems.filter(
                selectedItem => selectedItem.cartItemId !== item.cartItemId);
        },
        confirmCheckout() {
            this.isCheckoutDialogVisible = true;
        },
        async proceedToOrder() {
            this.isCheckoutDialogVisible = false;
            try {
                const selectedCartItems = this.cartItems.filter(item => this.selectedItems.includes(item));
                const orderItems = selectedCartItems.map(item => ({
                    cartItemId: item.cartItemId,
                    orderPrice: item.productPrice,
                    quantity: item.quantity
                }));
                console.log('orderItems:', orderItems)
                const response = await this.requestCreateOrderToDjango({ items: orderItems });
                const orderId = response.orderId;
                console.log(orderId)
            } catch (error) {
                console.error('Order creation failed:', error);
            }
        },
        async fetchCartList() {
            try {
                const response = await this.requestCartListToDjango();
                this.cartItems = response;
            } catch (error) {
                console.error("Error fetching cart list:", error);
            }
        },
    },
    created() {
        this.fetchCartList();
    },
};
</script>