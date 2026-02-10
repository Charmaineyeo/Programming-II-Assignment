# Reading order csv
using (StreamReader sr = new StreamReader("orders.csv"))
{
    string? s = sr.ReadLine(); // skip header
    while ((s = sr.ReadLine()) != null)
    {
        // STEP 1: Find where "Items" column starts
        int commaCount = 0;
        int itemsStartIndex = 0;
        
        for (int i = 0; i < s.Length; i++)
        {
            if (s[i] == ',')
            {
                commaCount++;
                if (commaCount == 9) // After 9 commas = Items column
                {
                    itemsStartIndex = i + 1;
                    break;
                }
            }
        }
        
        // STEP 2: Split the line
        string firstPart = s.Substring(0, itemsStartIndex - 1);
        string itemsPart = s.Substring(itemsStartIndex);
        
        // STEP 3: Parse first part
        string[] firstColumns = firstPart.Split(',');
        
        // STEP 4: Remove quotes
        itemsPart = itemsPart.Trim('"');
        
        // Extract data
        int orderId = Convert.ToInt32(firstColumns[0]);
        string customerEmail = firstColumns[1].Trim();
        string restaurantId = firstColumns[2].Trim();
        DateTime deliveryDateTime = Convert.ToDateTime(firstColumns[3] + " " + firstColumns[4]);
        string deliveryAddress = firstColumns[5].Trim();
        DateTime orderDateTime = Convert.ToDateTime(firstColumns[6]);
        double totalAmount = Convert.ToDouble(firstColumns[7]);
        string status = firstColumns[8].Trim();
        
        // ===== FIND CUSTOMER =====
        Customer foundCustomer = null;
        foreach (Customer c in customerList)
        {
            if (c.EmailAddress.Trim().ToLower() == customerEmail.ToLower())
            {
                foundCustomer = c;
                break;
            }
        }
        
        if (foundCustomer == null)
        {
            Console.WriteLine($"Customer {customerEmail} not found for order {orderId}.");
            continue;
        }
        
        // ===== FIND RESTAURANT =====
        Restaurant foundRestaurant = null;
        foreach (Restaurant r in restaurantList)
        {
            if (r.RestaurantId.Trim().ToLower() == restaurantId.ToLower())
            {
                foundRestaurant = r;
                break;
            }
        }
        
        if (foundRestaurant == null)
        {
            Console.WriteLine($"Restaurant {restaurantId} not found for order {orderId}.");
            continue;
        }
        
        // ===== CREATE ORDER =====
        Order order = new Order(orderId, orderDateTime, totalAmount, status, deliveryDateTime, deliveryAddress, "Credit Card", true, foundCustomer, foundRestaurant);
        
        // ===== ADD REAL ITEMS FROM CSV =====
        if (itemsPart != "")
        {
            string[] items = itemsPart.Split('|');
            
            foreach (string item in items)
            {
                string[] parts = item.Split(',');
                
                if (parts.Length >= 2)
                {
                    string itemName = parts[0].Trim();
                    int qty = Convert.ToInt32(parts[1].Trim());
                    
                    FoodItem foundFood = null;
                    foreach (FoodItem fi in foodItemList)
                    {
                        if (fi.ItemName.Trim().ToLower() == itemName.ToLower())
                        {
                            foundFood = fi;
                            break;
                        }
                    }
                    
                    if (foundFood != null)
                    {
                        double subTotal = foundFood.ItemPrice * qty;
                        OrderedFoodItem orderedItem = new OrderedFoodItem(
                            foundFood.ItemName,
                            foundFood.ItemDesc,
                            foundFood.ItemPrice,
                            "",
                            qty,
                            subTotal,
                            order
                        );
                        order.AddOrderedFoodItem(orderedItem);
                    }
                }
            }
        }
        
        // ===== SAVE ORDER =====
        orderList.Add(order);
        foundCustomer.orderList.Add(order);
        restaurantOrderQueue[foundRestaurant.RestaurantId].Enqueue(order);
    }
}

# Feature 6
//Part 6 - Luanne Lim
void ProcessOrder(Dictionary<string, Queue<Order>> restaurantOrderQueue,
                  List<Order> orderList,
                  Stack<Order> refundStack,
                  List<OrderedFoodItem> orderedFoodList)
{
    Console.WriteLine("Process Order");
    Console.WriteLine("=============");

    while (true)
    {
        Console.Write("Enter Restaurant ID (or X to exit): ");
        string inputRestaurantID = Console.ReadLine();

        if (inputRestaurantID.ToLower() == "x")
            break;

        // Check if restaurant exists
        if (!restaurantOrderQueue.ContainsKey(inputRestaurantID))
        {
            Console.WriteLine("Invalid Restaurant ID.\n");
            continue;
        }

        Queue<Order> orderQueue = restaurantOrderQueue[inputRestaurantID];

        if (orderQueue.Count == 0)
        {
            Console.WriteLine("No orders for this restaurant.\n");
            continue;
        }

        // Process orders one by one
        while (orderQueue.Count > 0)
        {
            Order detail = orderQueue.Peek(); // Just peek, don't dequeue yet!

            // Display order details
            Console.WriteLine($"\nOrder ID: {detail.OrderId}");
            Console.WriteLine($"Customer: {detail.Ordercustomer.CustomerName}");
            Console.WriteLine("Ordered Items:");

            if (detail.orderfooditemList == null || detail.orderfooditemList.Count == 0)
            {
                Console.WriteLine("No items in this order.");
            }
            else
            {
                int itemNum = 1;
                foreach (var item in detail.orderfooditemList)
                {
                    Console.WriteLine($"{itemNum}. {item.ItemName} - {item.QtyOrdered}");
                    itemNum++;
                }
            }

            // Display order total and status
            Console.WriteLine($"Total Amount: ${detail.CalculateOrderTotal():F2}");
            Console.WriteLine($"Delivery Date/Time: {detail.DeliveryDateTime: dd/MM/yyyy HH:mm}");
            Console.WriteLine($"Order Status: {detail.OrderStatus}");

            // Show options - FIXED: Added [O] option
            Console.Write("[C]onfirm / [R]eject / [O]ut for Delivery / [D]eliver / [S]kip: ");
            string choice = Console.ReadLine().ToLower();

            // Handle choice
            if (choice == "c")
            {
                if (detail.OrderStatus == "Pending")
                {
                    detail.OrderStatus = "Preparing";
                    Console.WriteLine($"Order {detail.OrderId} confirmed. Status: {detail.OrderStatus}");
                    // Remove from queue and add to the end
                    orderQueue.Dequeue(); // Remove the peeked order
                    orderQueue.Enqueue(detail); // Add it back with updated status
                }
                else
                {
                    Console.WriteLine($"Order {detail.OrderId} cannot be confirmed. Order brought back to the queue. Status: {detail.OrderStatus}");
                    // Cycle the order
                    orderQueue.Dequeue();
                    orderQueue.Enqueue(detail);
                }
            }
            else if (choice == "r")
            {
                if (detail.OrderStatus == "Pending")
                {
                    detail.OrderStatus = "Cancelled";
                    Console.WriteLine($"Order {detail.OrderId} rejected. Status: {detail.OrderStatus}");
                    refundStack.Push(detail); // Transfers to refund stack
                    orderQueue.Dequeue(); // Removed from order queue
                }
                else
                {
                    Console.WriteLine($"Order {detail.OrderId} cannot be rejected. Order brought back to the queue. Status: {detail.OrderStatus}");
                    // Cycle the order
                    orderQueue.Dequeue();
                    orderQueue.Enqueue(detail);
                }
            }
            else if (choice == "o") // This was missing in your original options display
            {
                if (detail.OrderStatus == "Preparing")
                {
                    detail.OrderStatus = "Out for Delivery";
                    Console.WriteLine($"Order {detail.OrderId} is out for delivery. Status: {detail.OrderStatus}");
                    // Cycle the order
                    orderQueue.Dequeue();
                    orderQueue.Enqueue(detail);
                }
                else
                {
                    Console.WriteLine($"Order {detail.OrderId} cannot be out for delivery. Order brought back to the queue. Status: {detail.OrderStatus}");
                    // Cycle the order
                    orderQueue.Dequeue();
                    orderQueue.Enqueue(detail);
                }
            }
            else if (choice == "d")
            {
                if (detail.OrderStatus == "Out for Delivery")
                {
                    detail.OrderStatus = "Delivered";
                    Console.WriteLine($"Order {detail.OrderId} delivered. Status: {detail.OrderStatus}");
                    orderQueue.Dequeue(); // Completely remove since delivered
                }
                else
                {
                    Console.WriteLine($"Order {detail.OrderId} cannot be delivered. Order brought back to the queue. Status: {detail.OrderStatus}");
                    // Cycle the order
                    orderQueue.Dequeue();
                    orderQueue.Enqueue(detail);
                }
            }
            else if (choice == "s")
            {
                Console.WriteLine($"Order {detail.OrderId} skipped.");
                // Just cycle the order without changing status
                orderQueue.Dequeue();
                orderQueue.Enqueue(detail);
            }
            else
            {
                Console.WriteLine("Invalid choice. Order returned to queue.");
                // Cycle the order on invalid input
                orderQueue.Dequeue();
                orderQueue.Enqueue(detail);
            }

            Console.WriteLine(); // Add spacing for readability
        }
    }
}
ProcessOrder(restaurantOrderQueue, orderList, refundStack, orderfooditemList);

# Classes
## Restuarant class:
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

## Fooditem Class:
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

## Order Class:
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

## Customer class:
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

## SpecialOffer class:
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

## Menu class:
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

## OrderdFoodItem class:
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

# Luanne's features:
## part 4
void DisplayallOrders(List<Order> orderList)
{
    Console.WriteLine("All Orders");
    Console.WriteLine("==========");
    Console.WriteLine("{0,-12}{1,-15}{2,-18}{3,-24}{4,-15}{5,-15}", "Order ID", "Customer", "Restaurant", "Delivery Date/Time", "Amount", "Status");
    Console.WriteLine("--------    -----------    -------------     ------------------      ------       ---------");
    foreach (Order order in orderList)
    {
        Console.WriteLine("{0,-12}{1,-15}{2,-18}{3,-25}{4,-12}{5,-10}", order.OrderId, order.Ordercustomer.CustomerName, order.Orderrestaurant.RestaurantName, order.DeliveryDateTime.ToString("yyyy-MM-dd HH:mm"), order.CalculateOrderTotal(), order.OrderStatus);
    }
}
//DisplayallOrders(orderList); output

## part 6
void Processorder(List<Order> orderList, List<OrderedFoodItem> orderedFoodList)
{
    Console.WriteLine("Process Order");
    Console.WriteLine("=============");

    while (true)
    {
        Console.Write("Enter Restaurant ID (or X to exit): ");
        string inputRestaurantID = Console.ReadLine();

        if (inputRestaurantID.ToLower() == "x")
        {
            break;
        }

        Console.WriteLine();

        bool found = false;

        foreach (Order detail in orderList)
        {
            if (inputRestaurantID == detail.Orderrestaurant.RestaurantId)
            {
                found = true;

                Console.WriteLine("Order {0}: ", detail.OrderId);
                Console.WriteLine("Customer: {0}", detail.Ordercustomer.CustomerName);
                Console.WriteLine("Ordered Items:");

                int itemnum = 1;

                foreach (OrderedFoodItem orderitem in orderedFoodList)
                {
                    if (detail.OrderId == orderitem.OrderOFI.OrderId)
                    {
                        Console.WriteLine("{0}. {1} - {2}",
                            itemnum, orderitem.ItemName, orderitem.QtyOrdered);
                        itemnum++;
                    }
                }

                Console.WriteLine("Delivery date/time: {0}", detail.DeliveryDateTime);
                Console.WriteLine("Total Amount: ${0}", detail.CalculateOrderTotal());
                Console.WriteLine("Order Status: {0}", detail.OrderStatus);

                Console.Write("[C]onfirm / [R]eject / [S]kip / [D]eliver: ");
                string userorderAnswer = Console.ReadLine().ToLower();

                if (userorderAnswer == "c" && detail.OrderStatus.ToLower() == "pending")
                {
                    detail.OrderStatus = "Preparing";
                }
                else if (userorderAnswer == "r" && detail.OrderStatus.ToLower() == "pending")
                {
                    detail.OrderStatus = "Rejected";
                }
                else if (userorderAnswer == "d" && detail.OrderStatus.ToLower() == "preparing")
                {
                    detail.OrderStatus = "Delivered";
                }
                else if (userorderAnswer == "s")
                {
                    continue; // skip order
                }
                else
                {
                    Console.WriteLine("Invalid action for current order status.");
                }

                Console.WriteLine("Order {0} updated. Status: {1}",
                    detail.OrderId, detail.OrderStatus);

                Console.WriteLine();
            }
        }

        if (!found)
        {
            Console.WriteLine("Restaurant ID is invalid. Please try again.\n");
        }
    }
}
//Processorder(orderList, orderfooditemList); 

## part 8
void DeleteExistingOrder(List<Order> orderList, List<OrderedFoodItem> orderedFoodList)
{
    Console.WriteLine("Delete Order");
    Console.WriteLine("============");

    Console.Write("Enter Customer Email:");
    string usercustomerEmail = Console.ReadLine().Trim().ToLower();

    Console.WriteLine("Pending Orders:");

    foreach (Order item in orderList)
    {
        if (item.OrderStatus.ToLower() == "pending" && item.Ordercustomer.EmailAddress.ToLower() == usercustomerEmail)
        {
            Console.WriteLine(item.OrderId);
        }
    }
    Console.Write("Enter Order ID: ");
    int userOrderID = Convert.ToInt32(Console.ReadLine());
    Console.WriteLine("\n");

    Order selectedOrder = null;

    foreach (Order item in orderList)
    {
        if (item.OrderId == userOrderID &&
            item.OrderStatus.ToLower() == "pending")
        {
            selectedOrder = item;
            break;
        }
    }

    if (selectedOrder == null)
    {
        Console.WriteLine("Order not found.");
        return;
    }
    Console.WriteLine("Customer: {0}", selectedOrder.Ordercustomer.CustomerName);
    Console.WriteLine("Ordered Items:");
    int itemnum = 1;

    foreach (OrderedFoodItem detail in orderfooditemList)
    {
        if (detail.OrderOFI.OrderId == selectedOrder.OrderId)
        {
            Console.WriteLine("{0}. {1} - {2}",
                itemnum, detail.ItemName, detail.QtyOrdered);
            itemnum++;
        }
    }

    Console.WriteLine("Delivery date/time: {0}", selectedOrder.DeliveryDateTime);
    Console.WriteLine("Total Amount: ${0}", selectedOrder.CalculateOrderTotal());
    Console.WriteLine("Order Status: {0}", selectedOrder.OrderStatus);

    Console.Write("Confirm deletion? [Y/N]: ");
    string userAnswer = Console.ReadLine().ToLower();

    if (userAnswer == "y")
    {
        selectedOrder.OrderStatus = "Cancelled";
        Console.WriteLine("Order {0} cancelled.", selectedOrder.OrderId);
    }
    else if (userAnswer == "n")
    {
        Console.WriteLine("Order not cancelled.");
    }
    else
    {
        Console.WriteLine("Invalid answer.");
    }
}
//DeleteExistingOrder(orderList, orderfooditemList);



// Question2 (Load files for Customers and Objects) -Charmaine
// load the customers.csv file and create customeer objects based on the data loaded
using (StreamReader sr = new StreamReader("customers.csv"))
{
    string? s = sr.ReadLine(); // read the heading line
    if (s != null)
    {
        string[] heading = s.Split(','); // displayb the headings
        //Console.WriteLine("{0,-16},{1,-16}", heading[0], heading[1]);
    }
    while ((s = sr.ReadLine()) != null)
    {
        string[] customerData = s.Split(',');
        string emailAddress = customerData[1];
        string customerName = customerData[0];
        Customer customer = new Customer(emailAddress, customerName);
        customerList.Add(customer);

        //Console.WriteLine(customer); // test output
    }
}
// load the orders.csv file and create order objects based on the data loaded
using (StreamReader sr = new StreamReader("orders.csv"))
{
    string? s = sr.ReadLine(); // skip header
    while ((s = sr.ReadLine()) != null)
    {
        string[] orderData = s.Split(',');

        int orderId = Convert.ToInt32(orderData[0]);
        string customerEmail = orderData[1].Trim();
        string restaurantId = orderData[2].Trim();
        DateTime deliveryDateTime = Convert.ToDateTime(orderData[3] + " " + orderData[4]);

        string deliveryAddress = orderData[5].Trim();
        DateTime orderDateTime = Convert.ToDateTime(orderData[6]);
        double totalAmount = Convert.ToDouble(orderData[7]);
        string status = orderData[8].Trim();

        // Find customer manually
        Customer foundCustomer = null;
        foreach (Customer c in customerList)
        {
            if (c.EmailAddress.Trim().ToLower() == orderData[1].Trim().ToLower())
            {
                foundCustomer = c;
                break;
            }
        }

        if (foundCustomer == null)
        {
            Console.WriteLine("Customer not found.");
            continue;
        }

        // Find restaurant manually
        Restaurant foundRestaurant = null;
        foreach (Restaurant r in restaurantList)
        {
            if (r.RestaurantId.Trim().ToLower() == orderData[2].Trim().ToLower())
            {
                foundRestaurant = r;
                break;
            }
        }

        if (foundRestaurant == null)
        {
            Console.WriteLine("Restaurant not found.");
            continue;
        }

        Order order = new Order(orderId, orderDateTime, totalAmount, status, deliveryDateTime, deliveryAddress, "Credit Card", true, foundCustomer, foundRestaurant);
        orderList.Add(order);
        // add order to customer's order list
        foundCustomer.orderList.Add(order);
        // add order to restaurant's order queue
        restaurantOrderQueue[foundRestaurant.RestaurantId].Enqueue(order);
        //Console.WriteLine(order); // test output
    }
}
