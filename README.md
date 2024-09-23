import ballerina/grpc;
import ballerina/log;

service class ShoppingService on new grpc:Listener(9090) {

    // In-memory store for products and carts
    map<Product> products = {};
    map<Product[]> carts = {};

    // Add product (admin)
    remote function add_product(AddProductRequest request) returns AddProductResponse {
        string sku = request.product.sku;
        products[sku] = request.product;
        log:printInfo("Product added: " + sku);
        return {sku: sku};
    }

    // Update product (admin)
    remote function update_product(UpdateProductRequest request) returns error? {
        string sku = request.sku;
        products[sku] = request.product;
        log:printInfo("Product updated: " + sku);
    }

    // Remove product (admin)
    remote function remove_product(RemoveProductRequest request) returns RemoveProductResponse {
        string sku = request.sku;
        products.remove(sku);
        log:printInfo("Product removed: " + sku);
        return {products: from map<Product> p in products select p};
    }

    // List available products (customer)
    remote function list_available_products() returns ListAvailableProductsResponse {
        Product[] availableProducts = from map<Product> p in products
                                      where p.available select p;
        return {products: availableProducts};
    }

    // Search product by SKU (customer)
    remote function search_product(SearchProductRequest request) returns SearchProductResponse {
        string sku = request.sku;
        Product? product = products[sku];
        if product is Product {
            return {product: product};
        } else {
            return {message: "Product not available"};
        }
    }

    // Add product to cart (customer)
    remote function add_to_cart(AddToCartRequest request) returns AddToCartResponse {
        Product? product = products[request.sku];
        if product is Product {
            carts[request.user_id].push(product);
            return {cart: carts[request.user_id]};
        } else {
            return error("Product not found");
        }
    }

    // Place order (customer)
    remote function place_order(PlaceOrderRequest request) returns PlaceOrderResponse {
        Product[] cart = carts[request.user_id];
        if cart.length() > 0 {
            string orderId = uuid:createType1AsString();
            carts.remove(request.user_id);
            return {order_id: orderId, message: "Order placed successfully"};
        } else {
            return {order_id: "", message: "Cart is empty"};
        }
    }

    // Create users (admin)
    remote function create_users(stream<CreateUsersRequest> userStream) returns CreateUsersResponse {
        log:printInfo("Creating users...");
        return {message: "Users created successfully"};
    }
}
