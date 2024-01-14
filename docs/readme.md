# Computer Architecture 2023: Term Project

[![hackmd-github-sync-badge](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw/badge)](https://hackmd.io/HdMEALKjTnSFF_d7QE3ESw)

:::success
## **Goal**
- [ ] 完成RISC-V Atom中所有examples的debug，確保所有example都能成功運行。這是首要目標，有助於驗證處理器的穩定性和正確性。
- [ ] 使用Verilator對RISC-V Atom進行驗證，確保支援Dhrystone等專案。這有助於確保處理器在實際應用中的正常運作。
- [ ] 嘗試實作RV32M指令集，並使用riscv-arch-test或riscv-tests進行全面的RV32I/RV32M指令測試，確保實作的指令集符合標準。
- [ ] 詳細記錄整個過程，包括遇到的技術問題。可以透過GitHub提交issue，與原開發者討論問題。參與整個過程並最終貢獻RV32M或相關實作將是一個更有價值的結果。

:::

## **Implement M extension**


To implement `M extension` in `riscv-atom`, we have to change the rtl code in `riscv-atom/rtl/core`. In [RISCV Instruction Set Manual](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf), you can find the description of the M extension, including the strategy for handling division by zero and overflow.

![image](https://hackmd.io/_uploads/B1bAlNxY6.png)

### Modify `Defs.vh` 
To include instructions for the `M extension`, different ALU operations are represented with four bits in the definition.

``` diff
// ALU
-`define ALU_FUNC_ADD    3'd0
-`define ALU_FUNC_SUB    3'd1
-`define ALU_FUNC_XOR    3'd2
-`define ALU_FUNC_OR     3'd3
-`define ALU_FUNC_AND    3'd4
-`define ALU_FUNC_SLL    3'd5
-`define ALU_FUNC_SRL    3'd6
-`define ALU_FUNC_SRA    3'd7
+`define ALU_FUNC_ADD    4'd0
+`define ALU_FUNC_SUB    4'd1
+`define ALU_FUNC_XOR    4'd2
+`define ALU_FUNC_OR     4'd3
+`define ALU_FUNC_AND    4'd4
+`define ALU_FUNC_SLL    4'd5
+`define ALU_FUNC_SRL    4'd6
+`define ALU_FUNC_SRA    4'd7


// ALU M extension
+`define ALU_FUNC_MUL    4'd8
+`define ALU_FUNC_MULH   4'd9
+`define ALU_FUNC_MULHSU 4'd10
+`define ALU_FUNC_MULHU  4'd11
+`define ALU_FUNC_DIV    4'd12
+`define ALU_FUNC_DIVU   4'd13
+`define ALU_FUNC_REM    4'd14
+`define ALU_FUNC_REMU   4'd15

```
### Modify `Decode.v` 



The instructions in the image below pertain to the `M extension`, and they all fall under the R-type category. To enable the decoder to understand these instructions, refer to the machine code definitions in the table for the corresponding RTL code
![image](https://hackmd.io/_uploads/rk5EFWWYa.png)



```diff
+            /* MUL   */
+            17'b0100001_000_0110011:
+            begin
+                instr_scope = "MUL";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MUL;
+            end
+            
+            /* MULH   */
+            17'b0100001_001_0110011:
+            begin
+                instr_scope = "MULH";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULH;
+            end
+            
+            /* MULHSU   */
+            17'b0100001_010_0110011:
+            begin
+                instr_scope = "MULHSU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULHSU;
+            end
+            
+            /* MULHU   */
+            17'b0100001_011_0110011:
+            begin
+                instr_scope = "MULHU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_MULHU;
+            end
+            
+            /* DIV   */
+            17'b0100001_100_0110011:
+            begin
+                instr_scope = "DIV";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_DIV;
+            end
+            
+            /* DIVU   */
+            17'b0100001_101_0110011:
+            begin
+                instr_scope = "DIVU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_DIVU;
+            end
+            
+            /* REM   */
+            17'b0100001_110_0110011:
+            begin
+                instr_scope = "REM";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_REM;
+            end
+            
+            /* REMU   */
+            17'b0100001_111_0110011:
+            begin
+                instr_scope = "REMU";
+                rf_we_o = 1'b1;
+                rf_din_sel_o = 3'd2;
+                a_op_sel_o = 1'b0;
+                b_op_sel_o = 1'b0;
+                alu_op_sel_o = `ALU_FUNC_REMU;
+            end
```
Also, change the `alu_op_sel` due to the adding one bit in `Defs.vh`
```diff=
-output reg [2:0] alu_op_sel_o,
+output reg [3:0] alu_op_sel_o,
```
### Modify `AtomRV.v`

change the wire `d-alu_op_sel` to 4 bit.
```diff
        ////// Instruction Decode //////
        Instruction decode unit decodes instruction and sets various control 
        signals throughout the pipeline. Is also extracts immediate values 
        from instructions and sign extends them properly.
    */
    wire    [4:0]   d_rd_sel;
    wire    [4:0]   d_rs1_sel;
    wire    [4:0]   d_rs2_sel;
    wire    [31:0]  d_imm;

    wire            d_jump_en;
    wire    [2:0]   d_comparison_type;
    wire            d_rf_we;
    wire    [2:0]   d_rf_din_sel;
    wire            d_a_op_sel;
    wire            d_b_op_sel;
    wire            d_cmp_b_op_sel;
-   wire    [2:0]   d_alu_op_sel;
+   wire    [3:0]   d_alu_op_sel;
    wire    [2:0]   d_mem_access_width;
    wire            d_mem_load_store;
    wire            d_mem_we;


```




### Modify `alu.v` 

Modify the input wire `sel_i` to be 4 bits and then integrate the circuitry to execute each instruction in the `M extension`

```diff
(
    input   wire    [31:0]  a_i,
    input   wire    [31:0]  b_i,
-   input   wire    [2:0]   sel_i,
+   input   wire    [3:0]   sel_i,

    output  reg     [31:0]  result_o
);

+/////// m extension
+    wire sel_mul 	= (sel_i == `ALU_FUNC_MUL);
+    wire sel_mulh	= (sel_i == `ALU_FUNC_MULH);
+    wire sel_mulhsu 	= (sel_i == `ALU_FUNC_MULHSU);
+    wire sel_mulhu 	= (sel_i == `ALU_FUNC_MULHU);
+    wire sel_div 	= (sel_i == `ALU_FUNC_DIV);
+    wire sel_divu 	= (sel_i == `ALU_FUNC_DIVU);
+    wire sel_rem 	= (sel_i == `ALU_FUNC_REM);
+    wire sel_remu 	= (sel_i == `ALU_FUNC_REMU);
+    /////// result of m extension
+    reg [63:0] mul_result;
+    reg [31:0] div_result;
+    reg [31:0] rem_result;
+     //mul,mulh,mulhsu,mulhu
+    always @(*) begin
+        if (sel_mul)
+            mul_result = $signed(a_i) * $signed(b_i);
+        else if (sel_mulhu)
+            mul_result = (a_i)*(b_i);
+        else if (sel_mulhsu)
+            mul_result = $signed(a_i)* (b_i);
+        else if (sel_mulh)
+            mul_result = $signed(a_i) * $signed(b_i);
+        else
+            mul_result= 64'h0  ;  
+    end
+    //div divu
+    always @(*) begin
+        if (sel_div)
+            div_result = (b_i == 32'h0) ? 32'hffffffff:
+            	   	  (a_i == 32'h80000000 && b_i == 32'hffffffff) ? 32'h80000000:
+                	  $signed(a_i) / $signed(b_i);
+        else if (sel_divu)
+            div_result = (b_i == 32'h0) ? 32'hffffffff:
+           		  $unsigned($unsigned(a_i) / $unsigned(b_i));
+        else
+            div_result= 32'h0;
+    end
+    //rem remu
+    always @(*) begin
+        if (sel_rem)
+            rem_result = (b_i == 32'h0) ? a_i:
+            	   	  (a_i == 32'h80000000 && b_i == 32'hffffffff) ? 32'h0:
+                	  $signed(a_i) % $signed(b_i);
+        else if (sel_remu)
+            rem_result = (b_i == 32'h0) ? a_i:
+            		  $unsigned($unsigned(a_i) % $unsigned(b_i));
+        else
+            rem_result= 32'h0;
+    end   


    // output of universal shifter
    reg [31:0] final_shift_output;
    always @(*) begin
        if (sel_sll)
            final_shift_output = reverse(shift_output[31:0]);
        else
            final_shift_output = shift_output[31:0];
    end
    
    // Final output mux
    always @(*) begin
        if (sel_add | sel_sub)
            result_o = arith_result;
        else if (sel_sll | sel_srl | sel_sra)
            result_o = final_shift_output;
        else if (sel_xor)
            result_o = a_i ^ b_i;
        else if (sel_or)
            result_o = a_i | b_i;
        else if (sel_and)
            result_o = a_i & b_i;
+        else if (sel_mul)
+            result_o = mul_result[31:0];
+        else if (sel_mulhu | sel_mulh | sel_mulhsu)
+            result_o = mul_result[63:32];
+        else if (sel_div | sel_divu)
+            result_o = div_result;
+        else if (sel_rem | sel_remu)
+            result_o = rem_result;
        else
            result_o = arith_result;
    end        
```


## Reference
* 