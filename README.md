import json
from cart import dao
from products import Product, get_product

class Cart:
    def _init_(self, id: int, username: str, contents: list[Product], cost: float):
        self.id = id
        self.username = username
        self.contents = contents
        self.cost = cost

    @staticmethod
    def load(data):
        return Cart(data['id'], data['username'], data['contents'], data['cost'])


def get_cart(username: str) -> list:
    """
    Retrieves the cart for the given username and returns a list of Product objects.
    """
    cart_details = dao.get_cart(username)
    if not cart_details:
        return []

    # Parse the cart contents once and batch-fetch product data
    items = []
    for cart_detail in cart_details:
        contents = json.loads(cart_detail['contents'])  # Using json.loads instead of eval
        items.extend(contents)

    # Batch fetch products to reduce individual database calls
    product_details = get_product_batch(items)
    return product_details


def add_to_cart(username: str, product_id: int):
    """
    Adds a product to the cart for the given username.
    """
    dao.add_to_cart(username, product_id)


def remove_from_cart(username: str, product_id: int):
    """
    Removes a product from the cart for the given username.
    """
    dao.remove_from_cart(username, product_id)


def delete_cart(username: str):
    """
    Deletes the cart for the given username.
    """
    dao.delete_cart(username)


def get_product_batch(product_ids: list[int]) -> list[Product]:
    """
    Batch fetches product details to minimize database calls.
    """
    return products.get_products_by_ids(product_ids)

    from products import dao


class Product:
    def _init_(self, id: int, name: str, description: str, cost: float, qty: int = 0):
        self.id = id
        self.name = name
        self.description = description
        self.cost = cost
        self.qty = qty

    @staticmethod
    def load(data: dict) -> "Product":
        """
        Loads a Product instance from a dictionary.
        """
        return Product(
            id=data['id'],
            name=data['name'],
            description=data['description'],
            cost=data['cost'],
            qty=data.get('qty', 0)  # Default to 0 if 'qty' is not provided
        )


def list_products() -> list[Product]:
    """
    Retrieves all products and returns a list of Product instances.
    """
    products_data = dao.list_products()
    return [Product.load(product) for product in products_data]  # Use list comprehension for better performance


def get_product(product_id: int) -> Product:
    """
    Retrieves a single product by its ID.
    """
    product_data = dao.get_product(product_id)
    if not product_data:
        raise ValueError(f"Product with ID {product_id} not found.")  # Add error handling
    return Product.load(product_data)


def add_product(product: dict):
    """
    Adds a new product using the provided dictionary data.
    """
    if 'id' not in product or 'name' not in product or 'cost' not in product:
        raise ValueError("Product data must include 'id', 'name', and 'cost'.")  # Validate input
    dao.add_product(product)


def update_qty(product_id: int, qty: int):
    """
    Updates the quantity of a product by its ID.
    """
    if qty < 0:
        raise ValueError("Quantity cannot be negative.")

    existing_product = dao.get_product(product_id)
    if not existing_product:
        raise ValueError(f"Product with ID {product_id} not found.")

    dao.update_qty(product_id, qty)









































































































































































