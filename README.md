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
