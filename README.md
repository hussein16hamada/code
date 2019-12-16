///// acvtivity code


public class AppBilling extends BaseActivity implements PurchasesUpdatedListener {
    BillingClient billingClient;
    RecyclerView recyclerView;
    Button load;
    MyAdapter myAdapter;
    List<SkuDetails>slist;

    public static final String pillPfrf = "pill";
    public static final String pillKey = "pill";
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_app_billing);

        setUpBillingClient();
        load=findViewById(R.id.load);

        recyclerView=findViewById(R.id.recycleView);
        recyclerView.setHasFixedSize(true);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        getThePlans();



    }

    private void getThePlans() {
        load.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (billingClient.isReady()){


                    SkuDetailsParams params= SkuDetailsParams.newBuilder()
                            .setSkusList(Arrays.asList("werd_monthly","werd_weekly"))
                            // sub or in app
                            .setType(BillingClient.SkuType.SUBS)
                            .build();

                    billingClient.querySkuDetailsAsync(params, new SkuDetailsResponseListener() {
                        @Override
                        public void onSkuDetailsResponse(BillingResult billingResult, List<SkuDetails> skuDetailsList) {
                            if (billingResult.getResponseCode()==BillingClient.BillingResponseCode.OK){

                                // handle the response as you want
                                 slist=skuDetailsList;
//                                for (SkuDetails skuDetails : skuDetailsList){
//
//                                }
                                loadProductToRecycleView();

//                                myAdapter.changeData(skuDetailsList);
                            }else {
                                Toast.makeText(AppBilling.this, "cant queryProduct", Toast.LENGTH_SHORT).show();

                            }
                        }
                    });


                }else {
                    Toast.makeText(AppBilling.this, "not ready", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }

    private void loadProductToRecycleView() {

         myAdapter=new MyAdapter(slist,this);
        recyclerView.setAdapter(myAdapter);

        myAdapter.setOnProcductClickListener(new MyAdapter.OnProcductClickListener() {
            @Override
            public void OnClicked(SkuDetails skuDetails, final int pos) {
                BillingFlowParams billingFlowParams= BillingFlowParams.newBuilder()
                        .setSkuDetails(skuDetails)
                        .build();

                billingClient.launchBillingFlow(activity,billingFlowParams);

            }
        });
    }

    private void setUpBillingClient() {
        billingClient= BillingClient.newBuilder(this).enablePendingPurchases().setListener(this).build();

        billingClient.startConnection(new BillingClientStateListener() {
            @Override
            public void onBillingSetupFinished(BillingResult billingResult) {
                if (billingResult.getResponseCode()==BillingClient.BillingResponseCode.OK){


//                    Toast.makeText(AppBilling.this, "success", Toast.LENGTH_SHORT).show();
                }else {
                    Toast.makeText(AppBilling.this, "fail"+billingResult.getResponseCode(), Toast.LENGTH_SHORT).show();

                }
            }

            @Override
            public void onBillingServiceDisconnected() {
                Toast.makeText(AppBilling.this, "disconnected", Toast.LENGTH_SHORT).show();

            }
        });



    }

    @Override
    public void onPurchasesUpdated(BillingResult billingResult, @Nullable List<Purchase> purchases) {
        if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.OK
                && purchases != null) {
            Toast.makeText(this, "تمت العمليه", Toast.LENGTH_SHORT).show();

            setIn(pillPfrf,pillKey,1);

        } else if (billingResult.getResponseCode() == BillingClient.BillingResponseCode.USER_CANCELED) {
            // Handle an error caused by a user cancelling the purchase flow.
            Toast.makeText(this, "العمليه لم تكتمل", Toast.LENGTH_SHORT).show();

        }

    }


}


//////adapter code

public class MyAdapter extends RecyclerView.Adapter<MyAdapter.myViewHolder> {

    List<SkuDetails> skuDetailsList;
    AppBilling mainActivity;

    OnProcductClickListener onProcductClickListener;


    public MyAdapter(List<SkuDetails> skuDetailsList, AppBilling mainActivity) {
        this.skuDetailsList = skuDetailsList;
        this.mainActivity = mainActivity;
    }
    public void setOnProcductClickListener(OnProcductClickListener onProcductClickListener) {
        this.onProcductClickListener = onProcductClickListener;
    }
    @NonNull
    @Override
    public myViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {

        View view = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.pilling_item, viewGroup, false);
        return new myViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull myViewHolder myViewHolder, final int i) {
        myViewHolder.product.setText(skuDetailsList.get(i).getDescription());

        final SkuDetails model=skuDetailsList.get(i);
        if (onProcductClickListener!=null){
            myViewHolder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    onProcductClickListener.OnClicked(model,i);
                }
            });
        }






    }

    @Override
    public int getItemCount() {
        if (skuDetailsList==null)return 0;
        return skuDetailsList.size();
    }
    public void changeData(List<SkuDetails> items) {
        this.skuDetailsList = items;
        notifyDataSetChanged();
    }
    public class myViewHolder extends RecyclerView.ViewHolder {
        TextView product;





        public myViewHolder(@NonNull View itemView) {
            super(itemView);
            product = itemView.findViewById(R.id.product);
        }


    }


    public interface OnProcductClickListener {
        void OnClicked(SkuDetails skuDetails, int pos);
    }


}

