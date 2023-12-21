---
title: fabric区块链(五)—解析basic智能合约（java）
date: 2023/5/20
tags: fabric区块链
categories: 区块链
top_img: ./img/41.jpg
cover: ./img/60.png
---

# 解析basic智能合约（java）：

### 首先，basic合约是我们之前在调用示例合约的时候调用的合约，fabric官方也提供了源码，在fabric/scripts/fabric-samples/asset-transfer-basic/chaincode-java/src/main/java/org/hyperledger/fabric/samples/assettransfer/目录下有AssertTransfer.java,Assert.java两个java类，逐一解析学习一下

![](./img/60.png)



### AssertTransfer.java

```go
/*
 * SPDX-License-Identifier: Apache-2.0
 */

package org.hyperledger.fabric.samples.assettransfer;

import java.util.ArrayList;
import java.util.List;


import org.hyperledger.fabric.contract.Context;
import org.hyperledger.fabric.contract.ContractInterface;
import org.hyperledger.fabric.contract.annotation.Contact;
import org.hyperledger.fabric.contract.annotation.Contract;
import org.hyperledger.fabric.contract.annotation.Default;
import org.hyperledger.fabric.contract.annotation.Info;
import org.hyperledger.fabric.contract.annotation.License;
import org.hyperledger.fabric.contract.annotation.Transaction;
import org.hyperledger.fabric.shim.ChaincodeException;
import org.hyperledger.fabric.shim.ChaincodeStub;
import org.hyperledger.fabric.shim.ledger.KeyValue;
import org.hyperledger.fabric.shim.ledger.QueryResultsIterator;

import com.owlike.genson.Genson;

@Contract(
        name = "basic",
        info = @Info(
                title = "Asset Transfer",
                description = "The hyperlegendary asset transfer",
                version = "0.0.1-SNAPSHOT",
                license = @License(
                        name = "Apache 2.0 License",
                        url = "http://www.apache.org/licenses/LICENSE-2.0.html"),
                contact = @Contact(
                        email = "a.transfer@example.com",
                        name = "Adrian Transfer",
                        url = "https://hyperledger.example.com")))
@Default
public final class AssetTransfer implements ContractInterface {

    private final Genson genson = new Genson();

    private enum AssetTransferErrors {
        ASSET_NOT_FOUND,
        ASSET_ALREADY_EXISTS
    }

    /**
     * Creates some initial assets on the ledger.
     *
     * @param ctx the transaction context
     */
    @Transaction(intent = Transaction.TYPE.SUBMIT)
    public void InitLedger(final Context ctx) {
        ChaincodeStub stub = ctx.getStub();

        CreateAsset(ctx, "asset1", "blue", 5, "Tomoko", 300);
        CreateAsset(ctx, "asset2", "red", 5, "Brad", 400);
        CreateAsset(ctx, "asset3", "green", 10, "Jin Soo", 500);
        CreateAsset(ctx, "asset4", "yellow", 10, "Max", 600);
        CreateAsset(ctx, "asset5", "black", 15, "Adrian", 700);
        CreateAsset(ctx, "asset6", "white", 15, "Michel", 700);

    }

    /**
     * Creates a new asset on the ledger.
     *
     * @param ctx the transaction context
     * @param assetID the ID of the new asset
     * @param color the color of the new asset
     * @param size the size for the new asset
     * @param owner the owner of the new asset
     * @param appraisedValue the appraisedValue of the new asset
     * @return the created asset
     */
    @Transaction(intent = Transaction.TYPE.SUBMIT)
    public Asset CreateAsset(final Context ctx, final String assetID, final String color, final int size,
        final String owner, final int appraisedValue) {
        ChaincodeStub stub = ctx.getStub();

        if (AssetExists(ctx, assetID)) {
            String errorMessage = String.format("Asset %s already exists", assetID);
            System.out.println(errorMessage);
            throw new ChaincodeException(errorMessage, AssetTransferErrors.ASSET_ALREADY_EXISTS.toString());
        }

        Asset asset = new Asset(assetID, color, size, owner, appraisedValue);
        // Use Genson to convert the Asset into string, sort it alphabetically and serialize it into a json string
        String sortedJson = genson.serialize(asset);
        stub.putStringState(assetID, sortedJson);

        return asset;
    }

    /**
     * Retrieves an asset with the specified ID from the ledger.
     *
     * @param ctx the transaction context
     * @param assetID the ID of the asset
     * @return the asset found on the ledger if there was one
     */
    @Transaction(intent = Transaction.TYPE.EVALUATE)
    public Asset ReadAsset(final Context ctx, final String assetID) {
        ChaincodeStub stub = ctx.getStub();
        String assetJSON = stub.getStringState(assetID);

        if (assetJSON == null || assetJSON.isEmpty()) {
            String errorMessage = String.format("Asset %s does not exist", assetID);
            System.out.println(errorMessage);
            throw new ChaincodeException(errorMessage, AssetTransferErrors.ASSET_NOT_FOUND.toString());
        }

        Asset asset = genson.deserialize(assetJSON, Asset.class);
        return asset;
    }

    /**
     * Updates the properties of an asset on the ledger.
     *
     * @param ctx the transaction context
     * @param assetID the ID of the asset being updated
     * @param color the color of the asset being updated
     * @param size the size of the asset being updated
     * @param owner the owner of the asset being updated
     * @param appraisedValue the appraisedValue of the asset being updated
     * @return the transferred asset
     */
    @Transaction(intent = Transaction.TYPE.SUBMIT)
    public Asset UpdateAsset(final Context ctx, final String assetID, final String color, final int size,
        final String owner, final int appraisedValue) {
        ChaincodeStub stub = ctx.getStub();

        if (!AssetExists(ctx, assetID)) {
            String errorMessage = String.format("Asset %s does not exist", assetID);
            System.out.println(errorMessage);
            throw new ChaincodeException(errorMessage, AssetTransferErrors.ASSET_NOT_FOUND.toString());
        }

        Asset newAsset = new Asset(assetID, color, size, owner, appraisedValue);
        // Use Genson to convert the Asset into string, sort it alphabetically and serialize it into a json string
        String sortedJson = genson.serialize(newAsset);
        stub.putStringState(assetID, sortedJson);
        return newAsset;
    }

    /**
     * Deletes asset on the ledger.
     *
     * @param ctx the transaction context
     * @param assetID the ID of the asset being deleted
     */
    @Transaction(intent = Transaction.TYPE.SUBMIT)
    public void DeleteAsset(final Context ctx, final String assetID) {
        ChaincodeStub stub = ctx.getStub();

        if (!AssetExists(ctx, assetID)) {
            String errorMessage = String.format("Asset %s does not exist", assetID);
            System.out.println(errorMessage);
            throw new ChaincodeException(errorMessage, AssetTransferErrors.ASSET_NOT_FOUND.toString());
        }

        stub.delState(assetID);
    }

    /**
     * Checks the existence of the asset on the ledger
     *
     * @param ctx the transaction context
     * @param assetID the ID of the asset
     * @return boolean indicating the existence of the asset
     */
    @Transaction(intent = Transaction.TYPE.EVALUATE)
    public boolean AssetExists(final Context ctx, final String assetID) {
        ChaincodeStub stub = ctx.getStub();
        String assetJSON = stub.getStringState(assetID);

        return (assetJSON != null && !assetJSON.isEmpty());
    }

    /**
     * Changes the owner of a asset on the ledger.
     *
     * @param ctx the transaction context
     * @param assetID the ID of the asset being transferred
     * @param newOwner the new owner
     * @return the old owner
     */
    @Transaction(intent = Transaction.TYPE.SUBMIT)
    public String TransferAsset(final Context ctx, final String assetID, final String newOwner) {
        ChaincodeStub stub = ctx.getStub();
        String assetJSON = stub.getStringState(assetID);

        if (assetJSON == null || assetJSON.isEmpty()) {
            String errorMessage = String.format("Asset %s does not exist", assetID);
            System.out.println(errorMessage);
            throw new ChaincodeException(errorMessage, AssetTransferErrors.ASSET_NOT_FOUND.toString());
        }

        Asset asset = genson.deserialize(assetJSON, Asset.class);

        Asset newAsset = new Asset(asset.getAssetID(), asset.getColor(), asset.getSize(), newOwner, asset.getAppraisedValue());
        // Use a Genson to conver the Asset into string, sort it alphabetically and serialize it into a json string
        String sortedJson = genson.serialize(newAsset);
        stub.putStringState(assetID, sortedJson);

        return asset.getOwner();
    }

    /**
     * Retrieves all assets from the ledger.
     *
     * @param ctx the transaction context
     * @return array of assets found on the ledger
     */
    @Transaction(intent = Transaction.TYPE.EVALUATE)
    public String GetAllAssets(final Context ctx) {
        ChaincodeStub stub = ctx.getStub();

        List<Asset> queryResults = new ArrayList<Asset>();

        // To retrieve all assets from the ledger use getStateByRange with empty startKey & endKey.
        // Giving empty startKey & endKey is interpreted as all the keys from beginning to end.
        // As another example, if you use startKey = 'asset0', endKey = 'asset9' ,
        // then getStateByRange will retrieve asset with keys between asset0 (inclusive) and asset9 (exclusive) in lexical order.
        QueryResultsIterator<KeyValue> results = stub.getStateByRange("", "");

        for (KeyValue result: results) {
            Asset asset = genson.deserialize(result.getStringValue(), Asset.class);
            System.out.println(asset);
            queryResults.add(asset);
        }

        final String response = genson.serialize(queryResults);

        return response;
    }
}

```

这段代码是一个基于Hyperledger Fabric框架的资产转移合约。让我逐行解释它的功能和结构。

首先，在代码开头的注释部分，指定了代码的许可证，这里使用的是Apache-2.0许可证。

接下来，代码定义了一个Java包`org.hyperledger.fabric.samples.assettransfer`，该包中包含了该合约的实现。

然后，代码开始定义了一个`AssetTransfer`类，并实现了`ContractInterface`接口。`ContractInterface`接口是Hyperledger Fabric合约的标准接口，合约需要实现这个接口才能在Fabric网络中运行。



接下来，使用了一些导入语句来引入所需的类和接口。

然后，使用`@Contract`注解标记了`AssetTransfer`类作为一个合约。`@Contract`注解提供了一些合约的元数据信息，如合约名称、描述、版本、许可证等。

在`AssetTransfer`类中，还使用了`@Transaction`注解来标记一个方法`InitLedger`作为一个交易函数。这意味着该方法将在提交交易时被调用。

`InitLedger`方法接受一个`Context`参数，用于访问链码（chaincode）的上下文。在方法中，通过`ctx.getStub()`获取了链码存根（ChaincodeStub）。

接下来，调用了`CreateAsset`方法来创建一些初始的资产。该方法接受`Context`和一些参数，用于创建一个新的资产。

至此，这段代码定义了一个基本的资产转移合约，包括合约的元数据信息、交易函数以及一些辅助方法。你提供的代码只是一个起点，后续可能会有更多的方法和逻辑来实现完整的资产转移功能。



### 然后解析学习一下Assert.java类

```
/*
 * SPDX-License-Identifier: Apache-2.0
 */

package org.hyperledger.fabric.samples.assettransfer;

import java.util.Objects;

import org.hyperledger.fabric.contract.annotation.DataType;
import org.hyperledger.fabric.contract.annotation.Property;

import com.owlike.genson.annotation.JsonProperty;

@DataType()
public final class Asset {

    @Property()
    private final String assetID;

    @Property()
    private final String color;

    @Property()
    private final int size;

    @Property()
    private final String owner;

    @Property()
    private final int appraisedValue;

    public String getAssetID() {
        return assetID;
    }

    public String getColor() {
        return color;
    }

    public int getSize() {
        return size;
    }

    public String getOwner() {
        return owner;
    }

    public int getAppraisedValue() {
        return appraisedValue;
    }

    public Asset(@JsonProperty("assetID") final String assetID, @JsonProperty("color") final String color,
            @JsonProperty("size") final int size, @JsonProperty("owner") final String owner,
            @JsonProperty("appraisedValue") final int appraisedValue) {
        this.assetID = assetID;
        this.color = color;
        this.size = size;
        this.owner = owner;
        this.appraisedValue = appraisedValue;
    }

    @Override
    public boolean equals(final Object obj) {
        if (this == obj) {
            return true;
        }

        if ((obj == null) || (getClass() != obj.getClass())) {
            return false;
        }

        Asset other = (Asset) obj;

        return Objects.deepEquals(
                new String[] {getAssetID(), getColor(), getOwner()},
                new String[] {other.getAssetID(), other.getColor(), other.getOwner()})
                &&
                Objects.deepEquals(
                new int[] {getSize(), getAppraisedValue()},
                new int[] {other.getSize(), other.getAppraisedValue()});
    }

    @Override
    public int hashCode() {
        return Objects.hash(getAssetID(), getColor(), getSize(), getOwner(), getAppraisedValue());
    }

    @Override
    public String toString() {
        return this.getClass().getSimpleName() + "@" + Integer.toHexString(hashCode()) + " [assetID=" + assetID + ", color="
                + color + ", size=" + size + ", owner=" + owner + ", appraisedValue=" + appraisedValue + "]";
    }
}

```

这段代码定义了一个名为`Asset`的Java类，用于表示资产。

首先，在代码开头的注释部分，指定了代码的许可证，这里使用的是Apache-2.0许可证。

然后，代码定义了一个名为`Asset`的类，并使用`@DataType`注解标记为合约数据类型。这个注解是Hyperledger Fabric框架的一部分，用于标识一个类是合约数据类型，可以在合约中使用。

接下来，使用了一些导入语句来引入所需的类和接口。

在`Asset`类中，定义了一些私有属性，使用`@Property`注解进行标记。这些属性包括`assetID`（资产ID）、`color`（颜色）、`size`（大小）、`owner`（所有者）和`appraisedValue`（评估值）。这些属性对应于资产的各个属性。

然后，定义了一系列的getter方法，用于获取属性的值。

接下来，定义了一个带有`@JsonProperty`注解的构造函数，用于根据给定的参数创建`Asset`对象。`@JsonProperty`注解用于指定属性与JSON字段之间的映射关系。

然后，重写了`equals`方法，用于比较两个`Asset`对象的相等性。在比较过程中，使用`Objects.deepEquals`方法比较了`assetID`、`color`和`owner`属性的值，以及`size`和`appraisedValue`属性的值。

这段代码定义了一个用于表示资产的类，包含了资产的各个属性以及相关的方法。该类在资产转移合约中被使用，用于创建和操作资产对象。
