# Programming-II-Assignment
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
            foreach(Order item in orderList)
            {
                if (item.OrderId == orderitem.OrderId)
                {
                    orderList.Add(orderitem);
                }
                else
                {
                    Console.WriteLine("Invalid OrderId");
                }
            }
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

# Food Item class:
namespace ASG_pt1
{
    internal class FoodItem
    {
        public string ItemName { get; set; }
        public string ItemDesc { get; set; }

        public double ItemPrice { get; set; }
        public string Customise { get; set; }

        public FoodItem(string name,string desc, double price, string customise)
        {
            ItemName = name;
            ItemDesc = desc;
            ItemPrice = price;
            Customise = customise;
            //note: remember to do the one or more thingy over here for the menu class
        }

        public override string ToString()
        {
            return "Item Name:" + ItemName + "\nItem Description: " + ItemDesc + "\nItem Price: " + ItemPrice + "\nCustomise: " + Customise;
        }
    }
}

# OrderedFoodItem Class:
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
            orderOFL.orderfooditemList.Add(this);
        }

        public double CalculateSubTotal()
        {
            return ItemPrice * QtyOrdered;
        }
    }
}

# Ordered Class:
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
        public List<OrderedFoodItem> orderfooditemList { get; set; } = new List<OrderedFoodItem> ();

        public Order() { }

        public Order(int orderid, DateTime orderdatetime, double ordertotal, string orderstatus, DateTime deliverydatetime, string deliveryaddress, string orderpaymentmethod, bool orderpaid, Customer customer)
        {
            OrderId = orderid;
            OrderDateTime = orderdatetime;
            OrderTotal = ordertotal;
            OrderStatus = orderstatus;
            DeliveryDateTime = deliverydatetime;
            DeliveryAddress = deliveryaddress;
            OrderPaymentMethod = orderpaymentmethod;
            OrderPaid = orderpaid;
            Ordercustomer = customer;

            customer.orderList.Add(this);
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

        }

        public override string ToString()
        {
            return base.ToString();
        }
    }
}
namespace S10273431_PRG2Assignment
{
    internal class Menu
    {
		private string menuId;

		public string MenuId
		{
			get { return menuId; }
			set { menuId = value; }
		}
		private string menuName;

		public string MenuName
		{
			get { return menuName; }
			set { menuName = value; }
		}
		// default constructor
		public Menu()
		{
        }
        // parameterized constructor
		public Menu(string menuId, string menuName)
		{
			this.menuId = menuId;
			this.menuName = menuName;
        }
		// method to add food item
		public void AddFoodItem(FoodItem foodItem)
		{

		}
    }
}
namespace S10273431_PRG2Assignment
{
    internal class Restaurant: Menu
    {
		private string restaurantId;

		public string RestaurantId
		{
			get { return restaurantId; }
			set { restaurantId = value; }
		}
		private string restaurantName;

		public string RestaurantName
		{
			get { return restaurantName; }
			set { restaurantName = value; }
		}
		private string restaurantEmail;

		public string RestaurantEmail
        {
			get { return restaurantEmail; }
			set { restaurantEmail = value; }
		}
		// default constructor
		public Restaurant()
		{
        }
        // parameterized constructor
		public Restaurant(string restaurantId, string restaurantName, string restaurantEmail)
		{
			this.restaurantId = restaurantId;
			this.restaurantName = restaurantName;
			this.restaurantEmail = restaurantEmail;
        }
		// method to display orders
		public void DisplayOrders()
		{

		}
        // method to display special offers
		public void DisplaySpecialOffers()
		{
        }
		// method to display menu
		public void DisplayMenu()
		{

		}
		// method to add menu 
		public void AddMenu(Menu menu)
		{
			
		}
        // method to remove menu
		public void RemoveMenu(Menu menu)
		{

		}
		public override string ToString()
		{
			return "Restaurant ID: " + restaurantId + "\nRestaurant Name: " + restaurantName + "\nRestaurant Email: " + restaurantEmail;
        }
    }
}

namespace S10273431_PRG2Assignment
{
    internal class SpecialOffer
    {
		private string offerCode;

		public string OfferCode
		{
			get { return offerCode; }
			set { offerCode = value; }
		}
		private string offerDesc;

		public string OfferDesc
		{
			get { return offerDesc; }
			set { offerDesc = value; }
		}
		private double discount;

		public double Discount
		{
			get { return discount; }
			set { discount = value; }
		}
		// default constructor
		public SpecialOffer()
		{
		}
		// parameterized constructor
		public SpecialOffer(string offerCode, string offerDesc, double discount)
		{
			this.offerCode = offerCode;
			this.offerDesc = offerDesc;
			this.discount = discount;
		}
		public override string ToString()
		{
			return "Offer Code: " + offerCode + "\nOffer Description: " + offerDesc + "\nDiscount: " + discount;
        }
    }
}
namespace S10273431_PRG2Assignment
{
    internal class Order
    {
        private List<OrderedFoodItem> orderedFoodItemList;

        private int orderId;

		public int OrderID
		{
			get { return orderId; }
			set { orderId = value; }
		}
		private DateTime orderDateTime;

		public DateTime OrderDateTime
		{
			get { return orderDateTime; }
			set { orderDateTime = value; }
		}
		private double orderTotal;

		public double OrderTotal
		{
			get { return orderTotal; }
			set { orderTotal = value; }
		}
		private string orderStatus;

		public string OrderStatus
		{
			get { return orderStatus; }
			set { orderStatus = value; }
		}
		private DateTime deliveryDateTime;

		public DateTime DeliveryDateTime
        {
			get { return deliveryDateTime; }
			set { deliveryDateTime = value; }
		}
		private string deliveryAddress;

		public string DeliveryAddress
		{
			get { return deliveryAddress; }
			set { deliveryAddress = value; }
		}
		private string orderPaymentMethod;

		public string OrderPaymentMethod
		{
			get { return orderPaymentMethod; }
			set { orderPaymentMethod = value; }
		}
		private bool orderPaid;

		public bool OrderPaid
		{
			get { return orderPaid; }
			set { orderPaid = value; }
		}
		// default constructor
		public Order() 
		{
			orderedFoodItemList = new List<OrderedFoodItem>();
        }
        // parameterized constructor
        public Order(int orderId, DateTime orderDateTime, double orderTotal, string orderStatus,
                     DateTime deliveryDateTime, string deliveryAddress, string orderPaymentMethod, bool orderPaid)
        {
            this.orderId = orderId;
            this.orderDateTime = orderDateTime;
            this.orderTotal = orderTotal;
            this.orderStatus = orderStatus;
            this.deliveryDateTime = deliveryDateTime;
            this.deliveryAddress = deliveryAddress;
            this.orderPaymentMethod = orderPaymentMethod;
            this.orderPaid = orderPaid;
			orderedFoodItemList = new List<OrderedFoodItem>();
        }
        // method to calculate order total
        public double CalculateOrderTotal()
		{
			double total = 0.0;
			foreach (OrderedFoodItem item in orderedFoodItemList)
			{
				total += item.SubTotal;
            }
			orderTotal = total;
			return total;
        }
        // method to add ordered food item
        public void AddOrderedFoodItem(OrderedFoodItem item)
        {
            orderedFoodItemList.Add(item);
        }
		// method to remove ordered food item
		public bool RemoveOrderedFoodItem(OrderedFoodItem item)
		{
			return orderedFoodItemList.Remove(item);
        }
        // method to display ordered food items
        public void DisplayOrderedFoodItems()
        {
            foreach (OrderedFoodItem item in orderedFoodItemList)
            {
                Console.WriteLine(item);
            }
        }
        public override string ToString()
		{
			return "Order ID: " + orderId + "Order DateTime: " + orderDateTime + "Order Total: " + orderTotal +
				   "Order Status: " + orderStatus + "Delivery DateTime: " + deliveryDateTime +
				   "Delivery Address: " + deliveryAddress + "Order Payment Method: " + orderPaymentMethod +
				   "Order Paid: " + orderPaid;
        }
    }
}

