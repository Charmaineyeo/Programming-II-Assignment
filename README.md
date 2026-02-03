# Restuarant class:
namespace ASG_pt1
{
    internal class Restaurant
    {
        public string RestaurantId { get; set; }
        public string RestaurantName { get; set; }
        public string RestaurantEmail { get; set; }
        public Restaurant MenuRestaurant { get; set; }

        // list to hold menus and special offers
        public List<Menu> menuList = new List<Menu>();
        public List<SpecialOffer> specialOfferList = new List<SpecialOffer>();
        public List<Order> orderList = new List<Order>();

        // default constructor
        public Restaurant()
        {
            menuList = new List<Menu>();
            specialOfferList = new List<SpecialOffer>();
            orderList = new List<Order>();
        }
        // parameterized constructor
        public Restaurant(string restaurantId, string restaurantName, string restaurantEmail)
        {
            RestaurantId = restaurantId;
            RestaurantName = restaurantName;
            RestaurantEmail = restaurantEmail;

            menuList = new List<Menu>();
            specialOfferList = new List<SpecialOffer>();
            orderList = new List<Order>();
        }
        // method to display orders
        public void DisplayOrders()
        {
            Console.WriteLine("Orders for " + RestaurantName);
            if (orderList.Count == 0)
            {
                Console.WriteLine("No orders yet.");
                return;
            }
            foreach (Order order in orderList)
            {
                Console.WriteLine(order);
                Console.WriteLine("Ordered Items:");
                order.DisplayOrderedFoodItems();
            }
        }
        // method to display special offers
        public void DisplaySpecialOffers()
        {
            Console.WriteLine("Special Offers for " + RestaurantName);
            foreach (SpecialOffer offer in specialOfferList)
            {
                Console.WriteLine(offer);
            }
        }
        // method to display menu
        public void DisplayMenu()
        {
            Console.WriteLine("Menus for " + RestaurantName);
            foreach (Menu menu in menuList)
            {
                menu.DisplayFoodItems();
            }
        }
        // method to add menu 
        public void AddMenu(Menu menu)
        {
            menuList.Add(menu);
        }
        // method to remove menu
        public void RemoveMenu(Menu menu)
        {
            menuList.Remove(menu);
        }
        public override string ToString()
        {
            return "Restaurant ID: " + RestaurantId + "\nRestaurant Name: " + RestaurantName + "\nRestaurant Email: " + RestaurantEmail;
        }
    }
 }

 # Fooditem Class:
 namespace ASG_pt1
{
    internal class FoodItem
    {
        public string ItemName { get; set; }
        public string ItemDesc { get; set; }

        public double ItemPrice { get; set; }
        public string Customise { get; set; }
        public Menu FoodItemMenu {  get; set; }

        public FoodItem(string name,string desc, double price, string customise)
        {
            ItemName = name;
            ItemDesc = desc;
            ItemPrice = price;
            Customise = customise;
        }

        public FoodItem(string name, string desc, double price, string customise, Menu menu): this(name, desc, price, customise)
        {
            FoodItemMenu = menu;
        }

        public override string ToString()
        {
            return "Item Name:" + ItemName + "\nItem Description: " + ItemDesc + "\nItem Price: " + ItemPrice + "\nCustomise: " + Customise;
        }
    }
}

# Order Class:
namespace ASG_pt1
{
    internal class Order
    {
        public int OrderId { get; set; }
        public DateTime OrderDateTime { get; set; }
        public double OrderTotal { get; set; }
        public string OrderStatus { get; set; }
        public DateTime DeliveryDateTime { get; set; }
        public string DeliveryAddress { get; set; }
        public string OrderPaymentMethod { get; set; }
        public bool OrderPaid { get; set; }
        public Customer Ordercustomer { get; set; }
        public Restaurant Orderrestaurant { get; set; }
        public SpecialOffer OrderspecialOffer { get; set; }
        public List<OrderedFoodItem> orderfooditemList { get; set; } = new List<OrderedFoodItem> ();

        public Order() { }

        public Order(int orderid, DateTime orderdatetime, double ordertotal, string orderstatus, DateTime deliverydatetime, string deliveryaddress, string orderpaymentmethod, bool orderpaid)
        {
            OrderId = orderid;
            OrderDateTime = orderdatetime;
            OrderTotal = ordertotal;
            OrderStatus = orderstatus;
            DeliveryDateTime = deliverydatetime;
            DeliveryAddress = deliveryaddress;
            OrderPaymentMethod = orderpaymentmethod;
            OrderPaid = orderpaid;
        }

        public Order(int orderid, DateTime orderdatetime, double ordertotal, string orderstatus, DateTime deliverydatetime, string deliveryaddress, string orderpaymentmethod, bool orderpaid, Customer customer, Restaurant restaurant)
            : this(orderid, orderdatetime, ordertotal,orderstatus, deliverydatetime, deliveryaddress, orderpaymentmethod, orderpaid)
        {
            Ordercustomer = customer;
            Orderrestaurant = restaurant;
        }

        public Order(int orderid, DateTime orderdatetime, double ordertotal, string orderstatus, DateTime deliverydatetime, string deliveryaddress, string orderpaymentmethod, bool orderpaid, Customer customer, Restaurant restaurant, SpecialOffer specialoffer)
            : this(orderid, orderdatetime, ordertotal, orderstatus, deliverydatetime, deliveryaddress, orderpaymentmethod, orderpaid, customer, restaurant)
        {
            OrderspecialOffer = specialoffer;
        }

        public double CalculateOrderTotal()
        {
            double orderTotal = 0;
            foreach(OrderedFoodItem item in orderfooditemList)
            {
                orderTotal += item.SubTotal;
            }
            return orderTotal;
        }

        public void AddOrderedFoodItem(OrderedFoodItem fooditem)
        {
           orderfooditemList.Add(fooditem);
        }

        public bool RemoveOrderedFoodItem(OrderedFoodItem fooditem)
        {
            orderfooditemList.Remove(fooditem);
            return true;
        }

        public void DisplayOrderedFoodItems()
        {
            foreach (OrderedFoodItem item in orderfooditemList)
            {
                Console.WriteLine(item);
            }
        }
        public override string ToString()
        {
            return "Order ID: " + OrderId + "Order DateTime: " + OrderDateTime + "Order Total: " + OrderTotal +
                   "Order Status: " + OrderStatus + "Delivery DateTime: " + DeliveryDateTime +
                   "Delivery Address: " + DeliveryAddress + "Order Payment Method: " + OrderPaymentMethod +
                   "Order Paid: " + OrderPaid;
        }
    }
}

# Customer class:
namespace ASG_pt1
{
    internal class Customer
    {
        public string EmailAddress { get; set; }
        public string CustomerName { get; set; }
        public List<Order> orderList {  get; set; } = new List<Order> ();
        public Customer() { }
        
        public Customer(string emailAddress, string customerName)
        {
            EmailAddress = emailAddress;
            CustomerName = customerName;
        }

        public void AddOrder(Order orderitem)
        {
            orderList.Add(orderitem);
        }

        public void DisplayAllOrders()
        {
            Console.WriteLine("All Orders:");
            foreach(Order item in orderList)
            {
                Console.WriteLine(item);
            }
        }

        public bool RemoveOrder(Order orderitem)
        {
            foreach (Order item in orderList)
            {
                if (item.OrderId == orderitem.OrderId)
                {
                    orderList.Remove(orderitem);
                    return true;
                }
                else
                {
                    Console.WriteLine("Invalid OrderId");
                }
            }
                return false;
        }

        public override string ToString()
        {
            return "Email Address: " + EmailAddress + "\tCustomer Name: " + CustomerName;
        }
    }
}

# SpecialOffer class:
namespace ASG_pt1
{
    internal class SpecialOffer
    {
        public string OfferCode { get; set; }
        public string OfferDesc { get; set; }
        public double Discount { get; set; }
        public Restaurant SOrestaurant { get; set; }

        public List<Order> orderList = new List<Order>();
        // default constructor
        public SpecialOffer(){ }
        // parameterized constructor
        public SpecialOffer(string offerCode, string offerDesc, double discount)
        {
            OfferCode = offerCode;
            OfferDesc = offerDesc;
            Discount = discount;
        }

        public SpecialOffer(string offerCode, string offerDesc, double discount, Restaurant restaurant): this(offerCode, offerDesc, discount)
        {
            SOrestaurant = restaurant;
        }
        public override string ToString()
        {
            return "Offer Code: " + OfferCode + "\nOffer Description: " + OfferDesc + "\nDiscount: " + Discount;
        }
    }
    }

# Menu class:
namespace ASG_pt1
{
	internal class Menu
	{
		public string MenuId { get; set; }
		public string MenuName { get; set; }
		public Restaurant MenuRestaurant { get; set; }

		// list to hold food items
		public List<FoodItem> foodItemList = new List<FoodItem>();

		// default constructor
		public Menu()
		{
			foodItemList = new List<FoodItem>();
		}
		// parameterized constructor
		public Menu(string menuId, string menuName)
		{
			MenuId = menuId;
			MenuName = menuName;
		}

		public Menu(string menuId, string menuName,Restaurant restaurant): this (menuId, menuName)
		{
			MenuRestaurant = restaurant;
		}

        // method to add food item
        public void AddFoodItem(FoodItem foodItem)
		{
			foodItemList.Add(foodItem);
		}
		// method to remove food item
		public bool RemoveFoodItem(FoodItem foodItem)
		{
			return foodItemList.Remove(foodItem);
		}
		// method to display all food items
		public void DisplayFoodItems()
		{
			Console.WriteLine("Food Items in Menu:");
			foreach (FoodItem item in foodItemList)
			{
				Console.WriteLine(item);
			}
		}
		public override string ToString()
		{
			return "Menu ID: " + MenuId + "\nMenu Name: " + MenuName;
		}
	}
}

# OrderdFoodItem class:
namespace ASG_pt1
{
    internal class OrderedFoodItem : FoodItem
    {
        public int QtyOrdered { get; set; }
        public double SubTotal { get; set; }
        public Order OrderOFI { get; set; }

        public OrderedFoodItem(string name, string desc, double price, string customise, int qtyOrdered, double subTotal, Order orderOFL) : base(name, desc,price, customise)
        {
            QtyOrdered = qtyOrdered;
            SubTotal = subTotal;
            OrderOFI = orderOFL;
        }

        public double CalculateSubTotal()
        {
            return ItemPrice * QtyOrdered;
        }
    }
}
