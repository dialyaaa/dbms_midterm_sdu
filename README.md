this is our code


1)  Procedure which does group by information 

  Сreating the shows_amount_spent procedure:

create or replace PROCEDURE shows_amount_spent
IS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Top buyer | Category   | Total spent ');
  DBMS_OUTPUT.PUT_LINE('--------- | ---------- | ----------- ');
  FOR lo IN(
    SELECT buyer.first_name, product_category.category_name, SUM(cart.quantity * product.price) AS total_amount
    FROM buyer, cart, order_details, product, product_category
    WHERE 
        buyer.buyer_id = cart.buyer_id AND
        cart.cart_id = order_details.cart_id AND
        cart.product_id = product.product_id AND
        product.category_id = product_category.category_id

    GROUP BY product_category.category_name, buyer.first_name
    ORDER BY total_amount DESC
  ) 
  LOOP
    DBMS_OUTPUT.PUT_LINE(
        RPAD(lo.first_name, 9) || ' | ' ||
        RPAD(lo.category_name, 10) || ' | ' ||
        RPAD(lo.total_amount || ' тг', 10) 
    );
  END LOOP;
END;

Calling the procedure:
	
BEGIN
shows_amount_spent;
END;


2) Function which counts the number of records:


DECLARE
  display_num NUMBER;
BEGIN
  display_num := count_students; 
  DBMS_OUTPUT.PUT_LINE('Number of products: ' || display_num); 
END;


3) Procedure which uses SQL%ROWCOUNT to determine the number of rows affected

CREATE OR REPLACE PROCEDURE update_quantity_of_products(
    q_product_id IN NUMBER,
    q_new_quantity IN NUMBER,
    q_message OUT VARCHAR2
) AS
    q_product_count NUMBER;
BEGIN
    SELECT COUNT(*) INTO q_product_count
    FROM product
    WHERE product_id = q_product_id;
    
    IF q_product_count = 0 THEN
        q_message := 'this product does not exist';
    ELSIF q_new_quantity < 0 THEN
        q_message := 'new value cannot be negative';
    ELSE
        UPDATE product
        SET quantity = q_new_quantity
        WHERE product_id = q_product_id;
        
        q_message := 'the quantity has been updated. affected number of rows: ' || SQL%ROWCOUNT;
    END IF;
END;

procedure call

DECLARE  
  v_message VARCHAR2(100);
BEGIN   
 update_quantity_of_products(3, 2, v_message);
    DBMS_OUTPUT.PUT_LINE(v_message);
    END;


4) Add user-defined exception which disallows to enter title of item (e.g. book) to be less than 5 characters

DECLARE 
   product_names product.product_name%type; 
   p_id product.product_id%type := 1; 
   lengthOfName NUMBER;
   ex_invalid_name EXCEPTION; 
BEGIN 
   SELECT product_name INTO product_names
   FROM product
   WHERE product_id = p_id and rownum=1;
   lengthOfName := LENGTH(product_names);
   
   IF lengthOfName <= 5 THEN 
      RAISE ex_invalid_name; 
   ELSE 
      DBMS_OUTPUT.PUT_LINE('Name of product: ' || product_names);  
   END IF; 
EXCEPTION 
   WHEN ex_invalid_name THEN 
      DBMS_OUTPUT.PUT_LINE('The number of letters in the name of product must be less than 5'); 
   WHEN no_data_found THEN 
      DBMS_OUTPUT.PUT_LINE('No data found!'); 
   WHEN others THEN 
      DBMS_OUTPUT.PUT_LINE('Error!');  
END;


5) Create a trigger before insert on any entity which will show the current number of rows in the table
Code of trigger:
create or replace TRIGGER count_rows
BEFORE INSERT ON product
FOR EACH ROW
DECLARE
  count_row NUMBER;
BEGIN
  SELECT COUNT(*) INTO count_row FROM product;
  DBMS_OUTPUT.PUT_LINE('Number of rows before insert in product table: ' || count_row);
END;




-Transaction:

Creating a procedure using a transaction:
create or replace PROCEDURE purchase_product (client_id IN NUMBER)
IS
  client_cart_id NUMBER;
  cl_product_id NUMBER;
  cl_quantity NUMBER;
  product_quantity NUMBER;
  cl_price NUMBER;
  client_balance NUMBER;
  cl_total_price NUMBER;
  test_bal number;
  test_qua number;
  ex exception;
  ex_balance exception;
BEGIN
  
  SELECT product_id INTO cl_product_id FROM cart WHERE buyer_id = client_id;

  SELECT balance INTO client_balance FROM card WHERE buyer_id = client_id;
  
  SELECT cart_id INTO client_cart_id FROM cart WHERE buyer_id = client_id;

  SELECT quantity INTO cl_quantity FROM cart WHERE product_id = cl_product_id;

  SELECT quantity INTO product_quantity FROM product WHERE product_id = cl_product_id;

  IF cl_quantity > product_quantity THEN
    RAISE ex;
  END IF;

  SELECT price INTO cl_price FROM product WHERE product_id = cl_product_id;

  cl_total_price := cl_quantity * cl_price;

  IF(cl_quantity <= product_quantity and client_balance>= cl_total_price) THEN
    UPDATE card SET balance = balance - cl_total_price WHERE buyer_id = client_id;
    SELECT balance into test_bal from card where buyer_id = client_id and rownum=1;
    dbms_output.put_line('Your balance after shopping - ' || test_bal);
    INSERT INTO order_details (status_of_payment, order_detalis_id, cart_id) VALUES ('true', 99999, client_cart_id);
    UPDATE product SET quantity = quantity - cl_quantity WHERE product_id = cl_product_id;
    SELECT quantity into test_qua from product where product_id = cl_product_id;
    dbms_output.put_line('Product quantity after shopping - ' || test_qua);
  ELSE
    RAISE ex_balance;
  END IF;
  COMMIT;
EXCEPTION
  WHEN ex THEN
    dbms_output.put_line('The product you have selected has run out or you have selected more than the quantity of the product');
 WHEN ex_balance THEN
    dbms_output.put_line('Your balance is not enough');
 WHEN OTHERS THEN
    ROLLBACK;
 END;

Calling the procedure:

begin 
purchase_product(21);
end;



